---
layout: post
title: NMF Topic Visualization with pyLDAvis (VW Scandal)
description: How to use pyLDAvis with a non-negative matrix factorization topic extraction model.
tags: [nmf, pyldavis, topic extraction, visualization, ]
comments: True
---
Recently I completed a project to extract topics from articles related to the Volkswagen Diesel Scandal. The project utilized Graphlab to model topics and [pyLDAvis](https://pyldavis.readthedocs.org/en/latest/){:target="_blank"} for visualization. As a follow on to the original project, I wanted to see if I could use pyLDAvis with non-negative matrix factorization (NMF) model for topic extraction. 

The following code snippet below assumes we have instantiated a vector over our corpus and fitted NMF model. See link to working visualization below.

{% highlight python %}
################################################
# Prepare data from NMF and Tfidf for pyLDAvis
################################################

# topic_term_dists
topic_term_dists = nmf.components_  # from NMF model

# normalize topic term vectors required by pyLDAvis
for topic_idx, topic in enumerate(topic_term_dists): 
    topic_term_dists[topic_idx] = topic_term_dists[topic_idx]/np.linalg.norm(topic_term_dists[topic_idx], ord=1)

if topic_term_dists.shape[0] != int(pd.DataFrame(topic_term_dists).sum(axis=1).map(lambda x: round(x, 2)).sum()):
    raise ValueError('Error - problem with topic term vector normalization')

# doc_topic_dists
doc_topic = []

# iterate through all docs in tfidf vector
for doc_idx, doc in enumerate(v_model_array):
    doc_topic_sims = []
    
    # iterate through all topics from NMF
    for topic_idx, topic in enumerate(nmf.components_):  
        
        # calculate similarity of topic to document
        sim_value = cosine_similarity(doc, topic)        
        doc_topic_sims.append(sim_value)

    doc_topic.append(doc_topic_sims)
    
doc_topic_dists = np.array(doc_topic)

# normalize topic features for each doc required by pyLDAvis
for doc_idx, doc in enumerate(doc_topic_dists):
    doc_topic_dists[doc_idx] = doc_topic_dists[doc_idx]/np.linalg.norm(doc_topic_dists[doc_idx], ord=1)

if doc_topic_dists.sum() != len(v_model_array):
    raise ValueError('Error - problem with document topic vector normalization')

# doc_lengths
doc_lengths = np.sum(v_model_array, axis=1)
    
# vocabulary
vocabulary = np.array(v_model_features)

# term_frequency
term_frequency = np.sum(v_model_array, axis=0)

pyLDAvis_data = {'topic_term_dists': topic_term_dists, 
                 'doc_topic_dists':  doc_topic_dists,
                 'doc_lengths':      doc_lengths,
                 'vocab':            vocabulary,
                 'term_frequency':   term_frequency}
import pyLDAvis
pyLDAvis.enable_notebook()

vis_data = pyLDAvis.prepare(**pyLDAvis_data)


# Display the interative visualization
pyLDAvis.display(vis_data)

{% endhighlight %}


Resulting pyLDAvis topic visualization below. Find the entire working notebook here: [pyLDAvis html interactive notebook](/notebooks/pyLDAvis-on-NMF.html#NMF-Topic-Visualization-with-pyLDAvis){:target="_blank"}.
<figure><a target="_blank" href="/notebooks/pyLDAvis-on-NMF.html#NMF-Topic-Visualization-with-pyLDAvis"><img src="/images/pyldavis.png"></a>
<figcaption></figcaption>
</figure>

Thanks to Ben Mabey for [BYOM](http://nbviewer.ipython.org/github/bmabey/pyLDAvis/blob/master/notebooks/pyLDAvis_overview.ipynb){:target="_blank"} (bring your own model) information.







