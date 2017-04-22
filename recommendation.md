# Recommending Notebooks within nbgallery
{:.no_toc}

Several members of our team had experience from previous projects with user-submitted code, so we anticipated that code duplication could be a potential problem.  To help minimize duplication, we’ve put a lot of effort into discoverability of notebooks - full-text search, tagging, useful metrics for sorting, and our [recommender system](https://en.wikipedia.org/wiki/Recommender_system).

* TOC
{:toc}

# Building blocks.

First let’s talk about some of the components that feed into our recommender system.

## User similarity

This measures how similar two users are based on the notebooks they’ve viewed.  To compute this, we first build a vector to represent each user’s activity in the Gallery.  Each user’s vector contains an entry for every notebook, with v[notebook id] = log(number of times they’ve viewed the notebook in the last 90 days) + (bonus 1.0 if the user has starred the notebook).  The 90-day cut-off helps account for changes in the user’s current duties, interests etc.  From eyeballing some of the vectors, typical values tend to fall between 0 and 4, with some outliers from notebook authors repeatedly viewing their own notebooks as they work on them.  After building the feature vectors, we use [cosine similarity](https://en.wikipedia.org/wiki/Cosine_similarity) to yield a score between 0 and 1 for each pair of users.

## Notebook similarity.

This measures how similar two notebooks are based on the content of the notebooks.  You can see the results of this computation in nbgallery in the “More Like This” section at the top of each notebook.

A common way to measure content similarity is to build a [TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf) matrix for the corpus and then use something like cosine similarity to compare the vectors for each document.  This was our initial approach as well, as we built our corpus from the code, markdown, title, and description of each notebook.  However, we ran into performance problems with the Ruby package we were using for the computation.  We believe this was at least in part because we included the code in the corpus, which was causing the number of terms to far exceed what it would be with just English text.  Our matrix was roughly 1000 documents by 75,000 terms, and the Ruby package often ran out of memory while processing it.  Fortunately, we are using [Apache Solr](http://lucene.apache.org/solr/) for our full-text search engine, and it provides a “more like this” function based on TF-IDF.  Although we don’t get a numeric score from Solr like we used to from running TF-IDF ourselves, it was fairly easy for us to switch over to it and the results are similar to our previous implementation.

We also worked with [Lab41](http://www.lab41.org/) on their [Altair challenge](https://github.com/Lab41/altair) for semantic understanding of code and measuring code similarity.  They explored a variety of algorithms, and one interesting quirk they found with the TF-IDF approach is a bias towards notebooks written by the same author - perhaps due to personal coding style and choice of variable names.  Despite this, they found that TF-IDF was more or less the state of the art for this problem.  They had some ideas for [improving the results with deep learning techniques](https://gab41.lab41.org/doc2vec-to-assess-semantic-similarity-in-source-code-667acb3e62d7), but unfortunately the challenge ended before they could investigate further.

## Users also viewed

This measures how similar two notebooks are based on what users are visiting them rather than the content of the notebooks.  You could see the results of this computation in nbgallery under the Users Also Viewed section at the top of each notebook.

We compute this using the [Jaccard index](https://en.wikipedia.org/wiki/Jaccard_index) on the sets of users that viewed each notebook.  We initially tried using raw view counts, but that caused various introductory and best-practice notebooks (which many users tend to view at least once) to end up on top.  We tried to correct for that but ended up promoting “rare” notebooks viewed by very few users.  The Jaccard index turned out to be much simpler to compute and provided a good balance between the two extremes.

## Trendiness

This measures a notebook’s popularity as a function of users views and time.  We compute this by adding up the number of unique users that viewed the notebook each day, multiplied by a scaling factor that decays from 1 for views that happened today down to 0 for views more than 30 days old.  We also factor in the age of the notebook itself to help prevent training notebooks from dominating the rankings.  We’ve made this the default sort when you browse all notebooks.

# Recommendation Algorithms

In our early discussions about how we would do personalized recommendations, we felt that we’d get better results if we combined different algorithms into a hybrid system.  We came up with several approaches, including traditional [collaborative](https://en.wikipedia.org/wiki/Recommender_system#Collaborative_filtering) and [content-based](https://en.wikipedia.org/wiki/Recommender_system#Content-based_filtering) techniques.

## Similar users

In a nutshell, our collaborative filtering algorithm looks at users that are similar to you in order to identify notebooks that are popular with them but less popular with you.  We start with the top N users most similar to you.  For each of them, we compare their feature vector with yours - the biggest differences are notebooks that they’ve viewed much more than you have.  Each notebook gets points based on how big that difference is, and we accumulate those points across the N similar users we look at.   The top scoring notebooks are saved in your recommendations.

## Similar Notebooks

Our content-based algorithm starts with the largest values in your feature vector - the notebooks you use the most.  For each of the top N notebooks, we use Solr's "more like this" search to get the most similar notebooks based on content.  As with the collaborative filtering algorithm, each notebook gets points based on how similar it is to the current notebook, and then we accumulate those points across your top N most-used notebooks.  The top scoring notebooks are saved in your recommendations. 

## Group membership

We use groups as a mechanism for shared ownership of notebook, but the group membership also makes a good signal for recommendations.  To do this, we identify notebooks that you haven’t viewed but are owned by a group you belong to.

## New Users

If you’ve only looked at a small number of notebook, then you’re probably a relatively new user.  In this case, we recommend some notebooks that are tagged as examples.

# Combining the Algorithms.

Each algorithm returns a recommendation score between 0 and 1 for a (user, notebook) pair.  We assigned some completely unscientific gut-feeling weights to each algorithm, but basically we just add up the scores of each notebook.  We then add in the trendiness score to give some extra weight to newer and more popular notebooks.  Finally, we throw out notebooks that you own or have shared (on the assumption that you’re already aware of them) as well as notebooks that you’ve viewed in the last few days.

So where do the recommendations show up in the Gallery?  First, if you browse all notebooks, you can sort by Relevance and the ones with the best aggregate recommendation score will bubble up to the top.  You’ll also see the reasons each one was recommended - e.g. “popular with similar users”.   Second, if you do a keyword search, we tell Solr to boost your recommendations in the results.  Solr obviously gives more weight to how well the notebook matches the search terms, but it will get a little bump if it’s also recommended for you.  Finally, there’s the Recommended For You page under the Notebooks menu.  Here we’ll show you a page of up to 20 notebooks.  Most of these are your top recommendations but we save a couple of slots at the bottom for randomly selected notebooks - we’re feeling lucky.

# Future Work

## Notebook Health

From our experience with other projects, a common reason for code duplication is that users are unsure if existing code still works.  Why worry about debugging someone else’s notebook when you can just write your own?  To address this, the latest version of our Jupyter image includes an instrumentation capability that will log the success or failure of code cell executions for all notebooks that originate from the Gallery.   After we gather and analyze some data, we plan to come up with some sort of “health score” that will give users an indication of whether a notebook is healthy or not.  Failed executions will penalize the cell, and each bad cell will penalize the overall notebook.  Execution logs will decay over time so that a notebook that hasn’t been run in several weeks will eventually show up with a unknown health instead of remaining at healthy.  Once we have a score algorithm we’re happy with, we’ll factor health scores into the recommender and use it to boost search results as well.
