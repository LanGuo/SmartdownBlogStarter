---
layout: blog
title: Experiment with text analysis
published: true
---

## Experiment with text analysis!



    ### This notebook is an experiment to scrape content from the web and do some semantic analysis with text data.
    - Scrapes customer reviews from a collagen supplement powder product on Amazon.com
    - stores the review (text string data) locally 
    - analyses the text data and attempt to isolate themes/topics 
    - produce simple visualization of results


    # -- Retrieve some AZ reviews for a collagen supplement product: -- #
    
    import  requests 
    from lxml import html
    from bs4 import BeautifulSoup
    
    url = 'https://www.amazon.com/Great-Lakes-Collagen-Hydrolysate-Grass-Fed/product-reviews/B01A1G47L0/ref=cm_cr_dp_see_all_summary/161-2449371-1192438?ie=UTF8&reviewerType=all_reviews&showViewpoints=1&sortBy=helpful'  #this is a popular collagen supplement powder
    
    
    headers = {'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.90 Safari/537.36'}
    
    
    page = requests.get(url, headers=headers) 
    documents = [] #list to store documents(individual reviews) in
    
    
    #doc = html.fromstring(page.content)
    #XPATH_REVIEWS = '//span[@data-hook="review-body"]//text()'
    #reviews = doc.xpath(XPATH_REVIEWS)  #This is a list of sentences from reviews
        
    # -- NEW method: use beautifulsoup for parsing html string and extracting review elements -- #
    soup = BeautifulSoup(page.content, 'html.parser')
    ## Beautiful Soup automatically converts incoming documents to Unicode and outgoing documents to UTF-8.
    for span in soup.findAll('span',attrs={"data-hook":"review-body"}):
        documents.append(span.get_text())
        
    # -- write reviews to txt file, notice they are in unicode encoding -- #    
    import codecs
    for s in documents:
        with codecs.open("/Users/Lan/Documents/kaggle/aztest.txt", "a", encoding="utf-8") as f:
            f.write(s+u'\n')  #write each sentence on a new line in txt file



    # -- load text back out and do theme/feature extraction -- #
    import codecs
    documents = codecs.open('/Users/Lan/Documents/kaggle/aztest.txt', encoding='utf-8').readlines()
    
    # -- Tokenize documents -- #
    # Having first installed nltk and downloaded nltk data:
    from nltk.tokenize import sent_tokenize  #This is the 'current recommended' sentence tokenizer in nltk
    #?sent_tokenize #to see which underlying model is this using:  currently :class:`.PunktSentenceTokenizer`
    sent_tokenize_list = [sent_tokenize(doc) for doc in documents]
    
    from nltk.tokenize import word_tokenize #This is the 'current recommended' word tokenizer in nltk
    # NOTE: word tokenizer kept the end '.' of each sentence as one token
    word_tokenize_per_doc = [word_tokenize(doc) for doc in documents]
    
    #word_tokenize_per_sent = [word_tokenize(sent) for sent in sent_tokenize_list] #keep sentence as individual documents
    #word_tokenize_list_all = word_tokenize(text) #lump all sentences into one document
    
    
    # -- Cleaning: Remove stop words and punctuation -- #
    from nltk.corpus import stopwords
    from nltk.stem.wordnet import WordNetLemmatizer
    import string
    stop = set(stopwords.words('english'))
    exclude_punc = set(string.punctuation)
    exclude_punc.update({'--','...'}) # After inspecting the output, found some nonconventional punctuations used in the review
    lemma = WordNetLemmatizer()
    
    stop_free_per_doc = [[word.lower() for word in doc if word.lower() not in stop] for doc in word_tokenize_per_doc] #remove common stop words
    punc_free_per_doc = [[word for word in doc if word not in exclude_punc] for doc in stop_free_per_doc] #remove punctuations
    
    # -- Cleaning: normalize/lemmatize words -- #
    #First time try to run this straight, got: UnicodeDecodeError: 'ascii' codec can't decode byte 0xe2 in position 0: ordinal not in range(128)
    #Seems like there are some weird non-ascii characters trialing some words or mixed in between words
    normalized_per_doc = []
    normalized_this_doc = []
    for doc in punc_free_per_doc:
        for word in doc:
            try:
                lemma.lemmatize(word) #lemmatize (normalize/get root of) words
                normalized_this_doc.append(word)
            except UnicodeDecodeError:
                print word #these are the weird non-ascii characters mixed into words, can ignore them or manually add the stem to normalized
        normalized_per_doc.append(normalized_this_doc)
        
    num_reviews = len(normalized_per_doc) #There were 10 reviews extracted
    
    
    #normalized_per_doc[0]




    [u'love',
     u'stuff',
     u'stir',
     u'morning',
     u'coffee',
     u'afternoon',
     u'yogurt',
     u'sometimes',
     u'evening',
     u'tea',
     u'mixes',
     u'well',
     u"'s",
     u'flavorless',
     u'ca',
     u"n't",
     u'tell',
     u"'s",
     u'digests',
     u'easily',
     u'aches',
     u'pains',
     u'fading',
     u'tons',
     u'reasons',
     u'gelatin',
     u'superfood',
     u'particular',
     ...]


    # -- Prepare document-term matrix -- #
    import gensim
    from gensim import corpora
    
    # Create the dictionary of our corpus, where every unique term(token) is assigned an index
    dictionary = corpora.Dictionary(normalized_per_doc)
    #dictionary.token2id
    
    # Convert tokenized documents to vectors using dict prepared above:
    doc_term_matrix = [dictionary.doc2bow(doc) for doc in normalized_per_doc]
    #print(doc_term_matrix[0])


    # -- Create LDA model -- #
    lda = gensim.models.ldamodel.LdaModel
    
    # Run and train lda model on doc_term_matrix
    #ldamodel = lda(doc_term_matrix,num_topics=2,id2word=dictionary,passes=20)
    
    ldamodel.print_topics(num_topics=2)
    
    # ! I did the normalized_per_doc wrong and all docs ended up with all the same words (all words in all docs)




    [(0,
      u'0.015*"\'s" + 0.013*"product" + 0.013*"collagen" + 0.011*"take" + 0.010*"hair" + 0.010*"great" + 0.009*"n\'t" + 0.008*"day" + 0.007*"container" + 0.007*"much"'),
     (1,
      u'0.002*"product" + 0.002*"collagen" + 0.002*"\'s" + 0.002*"n\'t" + 0.002*"great" + 0.002*"hair" + 0.002*"take" + 0.002*"taking" + 0.002*"one" + 0.002*"much"')]




    # **** 20161120 Night ~5:30-9:30pm **** #
    
    # -- Construct new documents corpus with each sentence in reviews forming an individual document -- #
    import codecs
    documents = codecs.open('/Users/Lan/Documents/kaggle/aztest.txt', encoding='utf-8').readlines()
    
    # -- Tokenize documents -- #
    from nltk.tokenize import sent_tokenize  #This is the 'current recommended' sentence tokenizer in nltk
    sent_tokenized_per_doc = [sent_tokenize(doc) for doc in documents]
    sent_tokenize_list = [item for sublist in sent_tokenized_per_doc for item in sublist]
    #sent_tokenize_list[5].split('...') #This sentence is a bit messed up...
    
    from nltk.tokenize import word_tokenize #This is the 'current recommended' word tokenizer in nltk
    # NOTE: word tokenizer kept the end '.' of each sentence as one token
    word_tokenize_per_sent = [word_tokenize(sent) for sent in sent_tokenize_list]
    
    # -- Cleaning: Remove stop words and punctuation -- #
    from nltk.corpus import stopwords
    from nltk.stem.wordnet import WordNetLemmatizer
    import string
    stop = set(stopwords.words('english'))
    stop.update({"'s","'t","n't","'m","'ve","per","'re","-i","\'\'","\u2014","still","much"}) #After inspecting the output, found some prevalent spelling of trivial words to remove
    exclude_punc = set(string.punctuation)
    exclude_punc.update({'--','...'}) # After inspecting the output, found some nonconventional punctuations used in the review
    lemma = WordNetLemmatizer()
    
    # From here on *doc* = one sentence
    stop_free_per_doc = [[word.lower() for word in sent if word.lower() not in stop] for sent in word_tokenize_per_sent] #remove common stop words
    punc_free_per_doc = [[word for word in sent if word not in exclude_punc] for sent in stop_free_per_doc] #remove punctuations
    
    # -- Cleaning: normalize/lemmatize words -- #
    #First time try to run this straight, got: UnicodeDecodeError: 'ascii' codec can't decode byte 0xe2 in position 0: ordinal not in range(128)
    #Seems like there are some weird non-ascii characters trialing some words or mixed in between words
    normalized_per_doc2 = [[lemma.lemmatize(word) for word in doc] for doc in punc_free_per_doc]
    print(normalized_per_doc2[10])
    
    # -- Prepare document-term matrix -- #
    import gensim
    from gensim import corpora
    
    # Create the dictionary of our corpus, where every unique term(token) is assigned an index
    dictionary2 = corpora.Dictionary(normalized_per_doc2)
    #dictionary.token2id
    
    # Convert tokenized documents to vectors using dict prepared above:
    doc_term_matrix2 = [dictionary2.doc2bow(doc) for doc in normalized_per_doc2] #This is also a 'corpus' in gensim: 
    #iterating over it yields each doc as its sparse vector representation
    print(doc_term_matrix[10])
    
    # -- Create LDA model -- #
    lda = gensim.models.ldamodel.LdaModel
    
    # Run and train lda model on doc_term_matrix
    n_topics = 10
    ldamodel2 = lda(doc_term_matrix2,num_topics=n_topics,id2word=dictionary2,passes=20)
    
    #ldamodel2.print_topics(num_topics=10)


    [u'still', u'cheaper', u'pound', u'company', u'website', u'looking', u'smaller', u'quantity', u'good', u'price.and', u'still', u'love']
    [(1, 1), (35, 1), (77, 2), (78, 1), (79, 1), (80, 1), (81, 1), (82, 1), (83, 1), (84, 1), (85, 1)]



    # -- Visualize topics --#
    # Code taken from https://gist.github.com/tokestermw/3588e6fbbb2f03f89798
    ## word lists
    for i in range(0, n_topics):
        temp = ldamodel2.show_topic(i, 10)
        terms = []
        for term in temp:
            terms.append(term)
        print "Top 10 terms for topic #" + str(i) + ": "+ ", ".join([i[0] for i in terms])
    
    
    


    Top 10 terms for topic #0: hair, face, take, happening, top, tbsp, week, started, something, look
    Top 10 terms for topic #1: product, container, yes, never, put, getting, ca, keep, lid, go
    Top 10 terms for topic #2: stuff, container, put, price, glass, one, plus, get, way, made
    Top 10 terms for topic #3: morning, product, coffee, take, gelatin, tea, day, mix, collagen, first
    Top 10 terms for topic #4: label, dissolve, every, product, green, investment, first, well, much, able
    Top 10 terms for topic #5: great, lake, collagen, best, buy, two, product, stuff, better, much
    Top 10 terms for topic #6: still, love, much, day, great, hair, open, twice, smaller, quantity
    Top 10 terms for topic #7: product, hair, would, month, nail, collagen, take, taking, better, thing
    Top 10 terms for topic #8: brand, like, collagen, get, little, might, way, well, huge, even
    Top 10 terms for topic #9: collagen, month, one, need, take, knee, better, look, away, work



    # -- Visualize topics --#
    # Code taken from https://gist.github.com/tokestermw/3588e6fbbb2f03f89798
    %matplotlib inline
    ## word clouds
    from os import path
    import matplotlib.pyplot as plt
    from wordcloud import WordCloud
    
    def terms_to_wordcounts(terms, multiplier=1000):
        return  " ".join([" ".join(int(multiplier*i[1]) * [i[0]]) for i in terms])
    
    i=5
    temp = ldamodel2.show_topic(i, 10)
    terms = []
    for term in temp:
         terms.append(term)
    wordcloud = WordCloud(background_color="black").generate(terms_to_wordcounts(terms))
    plt.imshow(wordcloud)
    plt.axis("off")
    plt.title('Topic #{}'.format(str(i)))
    
    





    <matplotlib.text.Text at 0x114467890>




![png](AMZN%20collagen%20review%20text%20analysis_files/AMZN%20collagen%20review%20text%20analysis_7_1.png)



    i=6
    temp = ldamodel2.show_topic(i, 10)
    terms = []
    for term in temp:
         terms.append(term)
    wordcloud = WordCloud(background_color="black").generate(terms_to_wordcounts(terms))
    plt.imshow(wordcloud)
    plt.axis("off")
    plt.title('Topic #{}'.format(str(i)))
    





    <matplotlib.text.Text at 0x11680c750>




![png](AMZN%20collagen%20review%20text%20analysis_files/AMZN%20collagen%20review%20text%20analysis_8_1.png)



    i=7
    temp = ldamodel2.show_topic(i, 10)
    terms = []
    for term in temp:
         terms.append(term)
    wordcloud = WordCloud(background_color="black").generate(terms_to_wordcounts(terms))
    plt.imshow(wordcloud)
    plt.axis("off")
    plt.title('Topic #{}'.format(str(i)))





    <matplotlib.text.Text at 0x115930810>




![png](AMZN%20collagen%20review%20text%20analysis_files/AMZN%20collagen%20review%20text%20analysis_9_1.png)



    


Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.
