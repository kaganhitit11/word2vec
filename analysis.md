1) Report and analyze the results obtained from different tokenization strategies. (10) 

I have tried each tokenizer with different training settings. Therefore, I thought it could be easier to track the results from Google Sheet. All of my experiment results can be found in the following Google Sheets link:
https://docs.google.com/spreadsheets/d/1mMsfiyl4dR6Q5RgfLHaDvKzJUMxwpOMKSHwY18SgYoc/edit?usp=sharing
P.S.: The Google Sheets document will not be updated after 11.59 p.m.,  23th of March 2024.

Word Tokenizer:

Two different trainings is done for word tokenizer. The only different hyperparameter between two trainings was batch size. In the first training, the batch size was 32 while the batch size was 128 in the second training. In the second training, I was able to achieve the syntactic accuracy as 0.0048 while the first training with batch size of 32 did not achieve well as it’s syntactic accuracy is 0.0.
As for the semantic analysis, training with batch size = 128 performed better than the training with batch size = 32 in all categories, especially for categories such as ‘aile, ülke-baskent’. However, both trainings did not perform well in categories such as ‘es-anlamlilar, zit-anlamlilar’ since relating a synonym pair is harder to relate than an ‘ülke-baskent’ pair by purely using continuous bag of words approach.

Character Trigram Tokenizer:

Two different trainings is done for character trigram tokenizer as well, the only different hyperparameter between two trainings being the context size. First training created the context by taking 2 neighboring left and right tokens of the target token for each token inside each training article. This way, the context size was 4 tokens for each target token. The second training created the context by taking 2 neighboring left and right words of the target word, also including the not-target tokens of the target word. This way, the context size was not fixed and was more than 4 tokens for each context vector, compared to the first context creation approach. In the first training with fixed context size of 2 tokens, the syntactic accuracy was 0.67. I consider this is because the model was able to capture the prefix-suffix differences of words with same roots since the context size was very small. On the other hand, 2-words context size training also performed very well as its semantic accuracy is 0.52. The syntacic MRR for the second training is 0.47.
As for semantic analysis, since the context size was very small for the 2-token context size training, the model was not able to capture any semantic relevance in analysis as the semantic accuracy is 0.0 for all categories. However, 2-word context size training was slightly better than the first training in categories such as ‘aile,’it has more number of tokens in the context vectors.

BPE Tokenizer:

Several trainings is done for the BPE tokenizer. First, two trainings are done with vocabulary size of 7K tokens each, context size = 2 tokens and batch size = 32. The difference between these two trainings was Turkish stopword removal. The training without stopword removal achieved syntactic accuracy of 0.50 while the training with stopword removal showed a fall with syntactic accuracy of 0.48. This decrease in syntactic accuracy may be attributed to the loss of essential grammatical and contextual information that stopwords provide. Then, the same training with stopword removal but with batch size of 512 is performed, and the syntactic accuracy dramatically fell to the level of around 0.35.
Then, as inspecting the tokens inside the vocabulary of my BPE tokenizer, I realized that the last couple thousands of tokens added to the vocabulary was mostly words with suffixes or prefixes. I considered this is a state to be avoided because adding many word-level tokens in a BPE tokenizer would eventually lead the tokenizer to tokenize the words as whole words like in the case of word tokenizer. Therefore, I reduced the vocabulary size to 4K tokens. Doing that, the syntactic accuracy for the 4K tokens and batch size 512 training remarkably increased from 0.35 to 0.43. I have also done another training with 4K tokens and batch size = 512, but changing the context size from 2 tokens to 4 tokens; but this did not change the syntactic accuracy of 0.43.

After all these fine-tuning of hyperparameters, I have done two more trainings, the first setup having 4K tokens of vocabulary size, batch size being 128, context size being 2 tokens, and stopword being applied. The second setup had batch size of 32 and context size of 2 words with padding applied, and the same collation logic is applied as in the character trigram case here. To remind the logic, take the 2 left and right neighboring words of the target word, and create context vector of the target word’s tokens using the neighboring words’ tokens and target word’s not-target tokens for each token of the target word. For both setups, the syntactic accuracy was 0.51, and the MRR was 0.45.

2) Discuss the impact of tokenization on model performance and any insights gained from the analogy tasks. (10) 

Word-level tokenization is not a very good idea for syntactic accuracy. Because in syntactic analysis, we determine the accuracy based on the model’s ability to capture the differences between words with same roots but with different suffixes or prefixes such as ‘aile – aileler – elma - elmalar’. In my training model, I was not able to achieve a word embedding that can capture the syntactic differences like that. The main reason I consider for that is the shortage of training articles (I trained word tokenizer with 150K due to lack of computation capacity). Because, as the model is fed with less amount of data, the less the embedding vectors are fine-tuned to capture the syntactic differences among the words with same roots.

Character trigram tokenization performs well in syntactic accuracy since we divide words to each possible subword with length 3 and use all of them in training. This way, the model is able to capture the prefixes and suffixes among words with same roots more accurately. 

This is further valid for BPE tokenizer if the tokenizer vocabulary is defined accordingly. Because, in BPE as discussed in the first question, if the vocabulary size is too large, BPE tends to work more like a word tokenizer since the last tokens generated by BPE can be word-level tokens. On the other side, if the vocabulary size is too small, BPE might tend to work more like a character trigram tokenizer since the longest tokens in BPE can be fixed to be 3 characters long. I think this is the strength of BPE in terms of allowing the model to perform well in terms of both syntactic and semantic accuracy. 

3) For your selected 10 words, report and analyze the 5 most similar words for each. 
words = ['gürültü', 'kitap','elma', 'genç', 'ifadeler','masa', 'banka', 'hekim','kurban','yaşarmış']

Word Tokenizer:
top_5_words_word_tokenizer =[
['gürültü', 'kapasitesine', 'atmaca', 'aydoğan', 'ömürleri']
['kitap', 'kitabı', 'grubun', 'grup', 'yazdı']
['elma', 'one', 'hakkında', 'köle', 'homo']
['genç', 'erkek', 'başarılı', 'iyi', 'kadın']
['ifadeler', 'ardında', 'şaka', 'verdiği', 'tanınan']
['masa', 'aslan', 'balina', 'yönünde', 'bulunur']
['banka', 'otoritesi', 'hyundai', 'güvenç', "avrupa'daki"]
['hekim', 'duvarlı', 'yunanlara', 'devlet', 'silahlı']
['kurban', 'olabilir', 'örneğin', 'izin', 'ifade']
['şşş', 'görülmüş', 'görülmktedir', 'görülmuş', 'görülmüslerdi']]

Character Trigram Tokenizer:
top_5_words_trigram_tokenizer = [[ ['gürültü', 'gürültülü', 'gürültüler', 'gürültüye', 'gürültüyü']
['kitap', 'kittap', 'kitaplı', 'kitapı', 'kitabı']
['elma', 'elmalma', 'elmaz', 'elmam', 'elmas']
['genç', 'gençli', 'gençlik', 'gençllik', 'genel']
['ifadeler', 'ifadelere', 'ifadeleri', 'ifadelerdi', 'ifadelerin']
['masa', 'makasa', 'makkasa', 'malasa', 'masar']
['banka', 'bankas', 'bankasi', 'bankai', 'bankası']
['hekim', 'hekimi', 'hekime', 'hekiminde', 'hekimli']
['kurban', 'turban', 'kurbanı', 'kurbana', 'kurbani']
['yaşarmış', 'yaşatmış', 'yaşarlarmış', 'yaşmış', 'yaşanmış']]

BPE Tokenizer:
top_5_words_bpe_tokenizer = [['gürültü', 'gürültücü', 'gürültüdür', 'gürültüleri', 'küçültü']
['kitap', 'kitapkurdu', 'yazıyorum', 'radyogrup', 'kitapcı']
['elma', 'mael', 'elmau', 'maetel', 'elmastır']
['genç', 'gençtürk', 'esirgenç', 'gençkızlar', 'şengenç']
['ifadeler', 'ifadeleri', 'ifadelerdir', 'işaretler', 'ifadelerle']
['sama', 'masa', 'masama', 'samama', 'masamachi']
['banka', 'kaban', 'kabanova', 'kabban', 'azkaban']
['hekim', 'marth', 'aralıkh', 'hekimhan', 'hekimoğlu']
['kurban', 'kurbantaşı', 'kurbanız', 'kurbanoğlu', 'kurbanlık']
['yaşarmış', 'yaşardığı', 'yaşarlarmış', 'yaşarmayı', 'yaşarmasına']]

The results show that character trigram and BPE is able to capture the similar words better than the word tokenizer. Also, in total, character trigram performs better than the BPE tokenizer since mostly all top-5 words are very similar to the target word.

4) Include any challenges faced during implementation and how they were overcome. (5) 

Handling huge amount of data in a compute cluster like Google Colab was something new to me. Therefore, I had hard time getting used to managing and processing the data more effectively. Especially, for the character trigram tokenizer training, I had hard time managing the system RAM of the Google Colab session in context-target pair generation since I was processing the data very ineffectively. I overcome these kind of computational issues by doing more in-place implementations for possible functions.

As I was new to PyTorch, I faced a learning curve in understanding its functionalities and syntax. To overcome this, I read a lot of PyTorch documentation and tutorials, which provided a strong foundation for working with the library effectively.

Also, generating context-target pairs for word2vec training presented me a significant challenge. There are various approaches to consider, including defining the window size and handling padding for sequences of different lengths. To achieve efficient data preparation, I invested time in researching different techniques and experimented with various padding methods as can be seen from other questions. 

5) Discuss your reasons for design choices such as context size, maximum number of subwords, preprocessing steps, minimum frequency for valid tokens, etc. (10) 

Handling OOV and PAD: For all tokenizers, I included <OOV> special token to the vocabulary for out-of-vocabulary tokens. For character trigram and BPE tokenizers, I also included the <PAD> special token to the vocabulary for possible paddings in context-target pair generation.

Word Tokenizer:

Preprocessing and Tokenizer Training: First, all articles bodies is collected in a list. All text is lowercased and split with whitespace. Then, text is filtered in character level so that the word tokens can only be composed of Turkish letters. All punctuation markes are also filtered out except the apostrophe; since in Turkish, there are words such as ‘Ela’, ‘Ela’yı’. Also, Turkish stop-words are removed from the text using the ‘turkce-stop-words.txt’ file, which is found inside the repository shared in the Discussion Board. Then, the most frequent 50K words are taken to be my vocabulary tokens for the word tokenizer, as it is suggested not to go beyond 50K tokens for any tokenizer in the homework description.

Context Size Choice: The context size for the word tokenizer is chosen to be 2 words since it is generally optimal for capturing immediate syntactic relationships without overcomplicating the model's learning process.
Corpus Size Choice: Due to lack of computation capacity, the corpus size is fixed and is the first 150K articles in the training data.

Learning Rate Choice: The learning rate is fixed to be 0.04 as the very first BPE training performed well in terms of syntactic accuracy. Therefore, I did not change the learning rate in any of my trainings. I’ve tried changing it to 0.08 in one of my trainings; however, I interrupted the training because the training loss was very volatile.
Batch Size Choice: I experimented trainings with batch sizes of 32 and 128, and the batch size of 128 performed better than batch size of 128 in terms of both analysis. 

Character Trigram Tokenizer: 

Preprocessing and Tokenizer Training: First, the same character filtering and stopword removal as the word tokenizer is applied to all training data. Then, character trigrams for all text is created and all character trigrams is added to my vocabulary. Since there is 32 possible characters (‘<’, ‘>’, Turkish letters, apostrophe), I expected the vocabulary size to be less than 50K, and it is indeed, approximately 27K.

Context Size Choice: Here, as stated in the first question, I trained my model twice with different context sizes: 2 words and 2 tokens. In 2 tokens approach, I generated each context vector by taking 2 left neighboring and 2 right neighboring tokens of the target token. This way, I did not have to pad the context vectors since they were all fixed size of 4. In 2 words approach, as mentinoed in the first question as well, I generated each context vector as follows: For each token inside each word, take the tokens of the target word’s 2 left neighboring words and 2 right neighboring words. Also, take the not-target tokens of target word’s tokens, and create the context vector of the target token using all these tokens, paying attention to the order of tokens. This led me to have context vectors with different sizes. Therefore, I used a collate function to pad the vectors using the <PAD> special token.
Corpus Size Choice: Due to lack of computation capacity, the corpus size is fixed and is first the 100K articles in the training data.

Learning Rate Choice: The learning rate is fixed to be 0.04 in all character trigram trainings as well.
Batch Size Choice: I fixed the batch size to be 128 in all trainings. I would not use a batch size smaller than 128 because that would take around 40 hours to train the model for 10 epochs. On the other hand, I did not want to use a batch size more than 128 because the syntactic analysis gave satisfactory results especially for my 2-tokens context size training (it was 0.67).

BPE Tokenizer:

Preprocessing and Tokenizer Training: First, the same character filtering and stopword removal as the word tokenizer is applied to all training data. Then, the ‘tokenizers’ library is used for BPE algorithm. Due to the nature of BPE algorithm, the vocabulary size of the tokenizer highly affects the training results. Therefore, I trained two separate tokenizers with vocab sizes 7K and 4K. I started with 7K, but visually inspecting the vocabulary, I realized that the tokens get very long after first 4K tokens. Therefore, I changed the vocabulary size to 4K tokens.

Context Size Choice: Here, I trained my model several times with different context sizes: 2 words, 2 tokens, and 4 tokens. In 2 tokens approach, I generated each context vector by taking 2 left neighboring and 2 right neighboring tokens of the target token. I applied exactly the same approach in 4 tokens approach. This way, I did not have to pad the context vectors since they were all fixed size. 
In 2 words approach, I generated each context vector applying the same logic as in the character trigram tokenizer’s 2 word approach. I used a collate function for padding of context vectors here, too.

6) Provide visualizations of some of the word embeddings and comment on them for each method. (10) 

7) Discuss ways of improving the Word2vec algorithm. (10)
 
Word2vec sometimes struggles with words that have multiple meanings. We can explore techniques like context-dependent embeddings or introducing a "sense" identifier to differentiate between the various meanings of a word. Also, rare words often get lost in the shuffle during training. We can use techniques that can help the model pay more attention to these infrequent words. Further, since Word2vec learns from the data it's fed, injecting external knowledge sources like ontologies or sentiment lexicons may potentially increase the accuracy of the embeddings, especially in terms of semantic analysis.
