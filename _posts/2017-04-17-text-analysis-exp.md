---
layout: blog
published: false
---
## Experiment with text analysis![AMZN collagen review text analysis_7_1.png]({{site.baseurl}}/media/AMZN collagen review text analysis_7_1.png)



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
     u'one',
     u'made',
     u'grass-fed',
     u'grass-finished',
     u'beef',
     u'serious',
     u'plus',
     u'book.but',
     u'happened',
     u'price',
     u'moment',
     u'get',
     u'six',
     u'cans',
     u'stuff',
     u'less',
     u'100',
     u'great',
     u'lakes',
     u'website',
     u"'s",
     u'16.16',
     u'per',
     u'including',
     u'shipping',
     u'opposed',
     u'current',
     u'37.89',
     u'per',
     u'34.45',
     u'buy',
     u'two',
     u'ca',
     u"n't",
     u'bring',
     u'fault',
     u'wonderful',
     u'product',
     u'price',
     u'gouging',
     u"'m",
     u'giving',
     u'fully',
     u'deserved',
     u'5',
     u'stars',
     u"'m",
     u'going',
     u'elsewhere',
     u'buy',
     u'price',
     u'comes',
     u'back',
     u'down.update',
     u'bought',
     u'first',
     u'product',
     u'last',
     u'november',
     u'writing',
     u'price',
     u'per',
     u'21.98',
     u'tad',
     u'less',
     u'paid',
     u'amazon',
     u'back',
     u"'s",
     u'still',
     u'cheaper',
     u'per',
     u'pound',
     u'company',
     u'website',
     u"'re",
     u'looking',
     u'smaller',
     u'quantity',
     u'good',
     u'price.and',
     u'still',
     u'love',
     u'aches',
     u'pains',
     u'gone',
     u'away',
     u'stayed',
     u'away',
     u'feeding',
     u'arthritic',
     u'cat',
     u'obviously',
     u'much',
     u'smaller',
     u'quantities',
     u'seems',
     u'helping',
     u'quite',
     u'bit',
     u'still',
     u'5-star',
     u'product',
     u'great',
     u'product',
     u'dissolves',
     u'easily',
     u'iced',
     u'tea.let',
     u'tell',
     u'little',
     u'dealing',
     u'first',
     u'anyone',
     u'else',
     u'might',
     u'help',
     u'-i',
     u'120',
     u'lbs',
     u'lose-i',
     u'3',
     u'severe',
     u'knee',
     u'injuries',
     u'ironically',
     u'working',
     u'-because',
     u'knee',
     u'injuries',
     u'also',
     u'developed',
     u'calcified',
     u'cyst',
     u'back',
     u'heel',
     u'well',
     u'plantar',
     u'fascitis-battle',
     u'low',
     u'thyroid',
     u'insulin',
     u'resistance',
     u'pcos-incredible',
     u'joint',
     u'pain',
     u'going',
     u'steps',
     u'walking',
     u'distanceafter',
     u'2',
     u'weeks',
     u'gl',
     u'collagen',
     u'noticed',
     u'following',
     u'changes-first',
     u'began',
     u'sleeping',
     u'way',
     u'night',
     u'never',
     u'happened',
     u'two',
     u'days',
     u'taking',
     u'it.-after',
     u'week',
     u'3',
     u'days',
     u'able',
     u'get',
     u'right',
     u'bed',
     u'without',
     u'pain',
     u'walked',
     u'stairs',
     u'like',
     u'normal',
     u'person',
     u"n't",
     u'even',
     u'know',
     u'huge',
     u'thing',
     u'able',
     u'exercise',
     u'everyday',
     u'without',
     u'needing',
     u'take',
     u'3-4',
     u'advil',
     u'every',
     u'night',
     u'due',
     u'knee',
     u'pain.-i',
     u'increased',
     u'energy',
     u'crazy',
     u'kind',
     u'steady',
     u'kind',
     u'supposed',
     u'have-i',
     u'able',
     u'kick',
     u'sugar',
     u'habit',
     u'energy',
     u'learn',
     u'cook',
     u'healthy',
     u'food.so',
     u'sum',
     u'huge',
     u'blessing',
     u'never',
     u'take',
     u'product',
     u'allowed',
     u'even',
     u'playing',
     u'field',
     u'fight',
     u'battle',
     u'obesity',
     u'follow',
     u'month',
     u'product',
     u'hope',
     u'helps',
     u'struggle',
     u'issues',
     u'battle',
     u'let',
     u'begin',
     u'saying',
     u'multiple',
     u'food',
     u'allergies',
     u'intolerances',
     u'makes',
     u'selection',
     u'consumable',
     u'products',
     u'limited',
     u'journey',
     u'great',
     u'lakes',
     u'products',
     u'began',
     u'knowledgable',
     u'helpful',
     u'company',
     u'representative',
     u'answered',
     u'questions',
     u'professionally',
     u'precisely',
     u'received',
     u'product',
     u'initially',
     u'pleased',
     u'quality',
     u'ease',
     u'use',
     u'plus',
     u'easy',
     u'dissolve',
     u'consume',
     u'using',
     u'twice',
     u'day',
     u'week',
     u'noticed',
     u'marked',
     u'improvement',
     u'arthritic',
     u'joints',
     u'two',
     u'weeks',
     u'arthritis',
     u'improved',
     u'95',
     u'month',
     u'bonus',
     u'hair',
     u'seems',
     u'better',
     u'\u2014',
     u'less',
     u'fine',
     u'strong',
     u'\u2014',
     u'even',
     u'skin',
     u'seems',
     u'better',
     u'fact',
     u'decreases',
     u'appetite',
     u'seems',
     u'calm',
     u'another',
     u'advantage',
     u"n't",
     u'expecting',
     u'though',
     u'bedtime',
     u'portion',
     u'relax',
     u'enough',
     u'able',
     u'go',
     u'right',
     u'sleep',
     u'recommend',
     u'product',
     u'great',
     u'lakes',
     u'bet',
     u'stuff',
     u'awesome',
     u"n't",
     u'realize',
     u'tried',
     u'another',
     u'brand',
     u'good',
     u'find',
     u'review',
     u'brand',
     u'profile',
     u'yes',
     u'pricey',
     u"'re",
     u'getting',
     u'specialty',
     u'product',
     u'ca',
     u"n't",
     u'find',
     u'places',
     u'want',
     u'cheap',
     u'gelatin',
     u'buy',
     u'knox',
     u'ew',
     u'right',
     u'moderate',
     u'degenerative',
     u'arthritis',
     u'hand',
     u'take',
     u'gelatin',
     u'help',
     u'manage',
     u'along',
     u'topical',
     u'ointment',
     u'instead',
     u'prescription',
     u'anti-inflammatory',
     u'pills',
     u'nasty',
     u'side',
     u'effects',
     u'stuff',
     u'keeps',
     u'pain',
     u'minimum',
     u'despite',
     u'working',
     u'computer',
     u'day',
     u'sorts',
     u'regular',
     u'daily',
     u'tasks.it',
     u'*immediately*',
     u'dissolves',
     u'hot',
     u'cold',
     u'liquid',
     u'big',
     u'plus',
     u"'ve",
     u'tried',
     u'brands',
     u'lumpy',
     u'gluey',
     u'nasty',
     u'gives',
     u'coffee',
     u'slightly',
     u'foamy',
     u'latte-like',
     u'froth',
     u'enjoy',
     u'mix',
     u'coconut',
     u'oil',
     u'milk',
     u'boom',
     u'morning',
     u'bliss',
     u'essentially',
     u'tasteless',
     u'coffee',
     u'tea',
     u'put',
     u'water',
     u'light-tasting',
     u'beverage',
     u'yes',
     u'taste',
     u'gelatin',
     u'mix',
     u'juice',
     u'something',
     u'else',
     u'might',
     u'know',
     u'there.it',
     u"'s",
     u'great',
     u'hair',
     u'nails',
     u'skin',
     u'claims',
     u'true',
     u'works.the',
     u'*small*',
     u'negative',
     u'lose',
     u'product',
     u'container',
     u'top',
     u"'s",
     u'rough',
     u'get',
     u'near',
     u'end',
     u'might',
     u'husband',
     u'mcgyver',
     u'way',
     u'get',
     u'open',
     u'put',
     u'glass',
     u'container',
     u'happens',
     u"'ll",
     u'post',
     u'update',
     u"'ll",
     u'sign',
     u'try',
     u'products',
     u'like',
     u'good',
     u'luck',
     u'never',
     u'without',
     u'great',
     u'lakes',
     u'**update',
     u'09/09/2015**',
     u'still',
     u'love',
     u'still',
     u'works',
     u'fact',
     u'figure',
     u'way',
     u'get',
     u'every',
     u'little',
     u'bit',
     u'yay',
     u'child',
     u'80s',
     u'like',
     u'might',
     u'remember',
     u'peach',
     u'hi-c',
     u'huge',
     u'mom',
     u'would',
     u'open',
     u'thing',
     u'2',
     u'sides',
     u'church',
     u'key',
     u'youngsters',
     u'non-southerners',
     u"'s",
     u'bottle',
     u'opener',
     u'curved',
     u'end',
     u'digress',
     u'used',
     u'church',
     u'key',
     u'open',
     u'top',
     u'pretty',
     u'much',
     u'peeled',
     u'away',
     u'got',
     u'open',
     u'wearing',
     u'gloves',
     u'could',
     u"n't",
     u'hurt',
     u'used',
     u'towel',
     u'poured',
     u'every',
     u'speck',
     u'resealable',
     u'glass',
     u'container',
     u'problem',
     u'solved',
     u'hope',
     u'helps',
     u'someone',
     u'**updated',
     u'11/13/2015**',
     u'hallelujah',
     u'great',
     u'lakes',
     u'heard',
     u'complaints',
     u'container',
     u'fixed',
     u'bought',
     u'2-pack',
     u'made',
     u'happy',
     u'discovery',
     u'take',
     u'top',
     u'plastic',
     u'lid',
     u'pull',
     u'tab',
     u'discard',
     u'pull',
     u'tab',
     u'cover',
     u'either',
     u'keep',
     u'container',
     u'plastic',
     u'lid',
     u'put',
     u'another',
     u'container',
     u'choice',
     u'use',
     u'glass',
     u'container',
     u'gasket',
     u'lid',
     u'keep',
     u'fresh',
     u'6-star',
     u'product',
     u'first',
     u'sure',
     u'buy',
     u'green',
     u'label',
     u'red',
     u'label',
     u'collagen',
     u'green',
     u'gelatin',
     u'red',
     u'wo',
     u"n't",
     u'dissolve',
     u'green',
     u'label',
     u'collagen',
     u'dissolves',
     u'tastes',
     u'put',
     u'anything',
     u'love',
     u'put',
     u"'healthy",
     u"coffee'",
     u'gano',
     u'cafe',
     u'3-in-1',
     u'gano',
     u'excel',
     u'usa',
     u'inc.',
     u'20',
     u'sachetsevery',
     u'morning',
     u'afternoon',
     u'one',
     u'two',
     u'tablespoons',
     u'day',
     u'recommended',
     u'ordered',
     u'collagen',
     u'doctor',
     u"'s",
     u'best',
     u'best',
     u'collagen',
     u'types',
     u'1',
     u'3',
     u'7.1',
     u'ounce',
     u'200-grams',
     u'compared',
     u'labels',
     u'cheapest',
     u'great',
     u'quality.price',
     u'holy',
     u'crap',
     u'site',
     u'22.19',
     u'found',
     u'else',
     u'9.81',
     u'go',
     u'great',
     u'lakes',
     u'website',
     u'get',
     u'several',
     u'cans',
     u'much',
     u'cheaper',
     u'one',
     u'go',
     u'supplement',
     u'warehouse.health',
     u'benefits',
     u'nd',
     u'recommended',
     u'product',
     u'sold',
     u'product',
     u'low',
     u'bone',
     u'density',
     u'would',
     u'forever',
     u'tendentious',
     u'reason',
     u'took',
     u'product',
     u'hurting',
     u'went',
     u'away',
     u'could',
     u'tell',
     u'tendons',
     u'getting',
     u'stronger',
     u'stopped',
     u'getting',
     u'injured',
     u'much',
     u'laid',
     u'three',
     u'years',
     u"n't",
     u'see',
     u'coming',
     u'strong',
     u'nails',
     u'became',
     u'never',
     u'nails',
     u'mention',
     u'strong',
     u'nails',
     u'stop',
     u'taking',
     u'collagen',
     u'nails',
     u'got',
     u'weaker',
     u'kind',
     u'nice',
     u'barometer',
     u'knowing',
     u"'s",
     u'happening',
     u'inside.my',
     u'hair',
     u'started',
     u'growing',
     u'like',
     u'crazy',
     u'much',
     u'better',
     u'condition',
     u'survivor',
     u'bad',
     u'hair',
     u'cut',
     u'rather',
     u'grateful',
     u'times',
     u'hair',
     u'would',
     u'dry',
     u'besides',
     u'torcher',
     u'hair',
     u'seemed',
     u'nothing',
     u'would',
     u'improve',
     u'hair',
     u'says',
     u'something',
     u'internal',
     u'happening',
     u'well',
     u'within',
     u'first',
     u'week',
     u'hair',
     u'started',
     u'growing',
     u'fast',
     u'longer',
     u'dry',
     u'hmmmm.skin',
     u'look',
     u'younger',
     u'yes',
     u'seriously',
     u'always',
     u'look',
     u"'s",
     u'happening',
     u'face',
     u'google',
     u"'chinese",
     u'face',
     u'reading',
     u'lines',
     u'face',
     u'health',
     u'internally',
     u'face',
     u'lines',
     u'come',
     u'go',
     u'unless',
     u'smoker',
     u'really',
     u'noticed',
     u'difference',
     u'eyes',
     u'neck',
     u'seller',
     u'arms',
     u"'s",
     u'thing',
     u'goin',
     u'look',
     u'much',
     u'younger',
     u'age',
     u'59',
     u'lucky',
     u'genes',
     u'watch',
     u'eating',
     u'sugar',
     u'take',
     u'collagen',
     u'simple',
     u'take',
     u'age',
     u'yeah',
     u'need',
     u'hair',
     u'thinning',
     u'started',
     u'taking',
     u'daily',
     u'3',
     u'weeks',
     u'already',
     u'see',
     u'difference.i',
     u'take',
     u'2',
     u'tbsp',
     u'1',
     u'tbsp',
     u'metamucil',
     u'fiber',
     u'suppose',
     u'take',
     u'twice',
     u'day',
     u"'ve",
     u'taking',
     u'once.it',
     u"'s",
     u'tolerable',
     u'drink',
     u'since',
     u'works',
     u'one',
     u'thing',
     u'note',
     u"'ve",
     u'never',
     u'may',
     u'need',
     u'start',
     u'low',
     u'1/4',
     u'tsp',
     u'day',
     u'work',
     u'way',
     u"'re",
     u'gas',
     u'pains',
     u'diarrhea',
     u'likely',
     u'need',
     u'lower',
     u'amount',
     u"'ve",
     u'taking',
     u'almost',
     u'four',
     u'months',
     u'besides',
     u'skin',
     u'hair',
     u'nail',
     u'improvement',
     u"'ve",
     u'seen',
     u'menstrual',
     u'cycle',
     u'improvement',
     u'idea',
     u'would',
     u'take',
     u'place',
     u'collagen',
     u'helps',
     u'balance',
     u'hormones',
     u'ladies',
     u'world',
     u'convinced',
     u'stuff',
     u'remedy',
     u'many',
     u'women',
     u"'s",
     u'biological',
     u'issues',
     u'began',
     u'using',
     u'product',
     u'soft',
     u'tissue',
     u'pcl',
     u'knee',
     u'injury',
     u'would',
     u'heal',
     u'period',
     u'six',
     u'months',
     u'physical',
     u'therapy',
     u'one',
     u'month',
     u'saw',
     u'dramatic',
     u'improvement',
     u'injury',
     u'four',
     u'months',
     u'twice-daily',
     u'doses',
     u'collagen',
     u'full',
     u'function',
     u'knee',
     u'part',
     u'daily',
     u'routine',
     u'reading',
     u'studies',
     u"'s",
     u'protect',
     u'investment',
     u'first',
     u'take',
     u'product',
     u'taking',
     u'nsaid',
     u'pain',
     u'relievers',
     u'advil',
     u'apparently',
     u'inhibit',
     u'body',
     u"'s",
     u'ability',
     u'absorb',
     u'collagen',
     u'next',
     u'may',
     u'consider',
     u'taking',
     u'extra',
     u'vitamin',
     u'c',
     u'take',
     u'one',
     u'dose',
     u'per',
     u'day',
     u'collagen',
     u'apparently',
     u'binds',
     u'vitamin',
     u'c',
     u'depletes',
     u'take',
     u'added',
     u'1000',
     u'mg',
     u"'s",
     u'day',
     u'500',
     u'mg',
     u'doses',
     u'collagen',
     u'morning',
     u'evening',
     u'finally',
     u'put',
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
