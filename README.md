# A simple named entity recognition custom model from scratch with annotation tool prodi.gy
Update: A new model is build with Spacy 3 on 12 November 2022.

Creation of virtual environment (prodigy-env) in ubuntu
Step. Open the folder in terminal, where you want to create virtual environment with (ctrl+alt+t)

Step. type in the terminal

python3 -m venv prodigy-env
Step. (You can see that there is prodigy-env in the folder you have opened in the terminal)

Step. How to open prodi.gy environment? Open the folder where your prodigy-env folder is and then type following:

source prodigy-env/bin/activate
Step. You will see that on the left side of the terminal that you are in the virtual environment

Start annotating with new text
Step. Download en_core_web_sm or en_core_web_lg model for annotating by typing 'pip install model_name' in the virtual environment.

Step. Keep your text ready in the folder where virtual environment is (this is not necessary if you know the path to file).

Step Type the command from prodigy website to annotate

prodigy ner.manual test_dataset en_core_web_lg /home/gebruiker/anaconda3/envs/test.txt - label PERSON,ORG,PRODUCT
where,

ner.manual is the recipe,

test_dataset is the annotation dataset (to be created),

en_core_web_lg is the Spacy model used to annotate,

/home/gebruiker/anaconda3/envs/test.txt is the path where your test.txt is for annotation and

— label PERSON,ORG,PRODUCT are the labels. You can add or change the labels as you like.

At the first run, prodigy creates prodigy.db and prodigy.json in the home folder.

Model creation
Step. Type following to create model:

prodigy train /home/gebruiker/Documenten/ - ner test_dataset
where,

train is the recipe,

/home/gebruiker/Documenten/ is the path where the model will be saved

— ner is to create NER model

test_dataset is the annotated dataset from previous step

Previous version of this article with detailed building of models with Spacy 2
# NER_model_prodigy

How to build a named entity recognition custom model with annotation tool prodi.gy

Note: the model was built last year June 2021 with Spacy 2.3.7. The codes are no longer up to date (august 2022). However this article gives a general idea how to build a model with prodi.gy.

SpaCy can be used to extract entities from a given text. You can use some of the available natural language models from spaCy to extract information such as personsname, organisation name or location from the text. The natural language processing (NLP) named entity recognition models such as en_core_web_lg are trained on wikipedia and on a number of web pages. However, the model does not recognize custom-specific entities such as name of animal species or an academic award. However if you want to have custom labels, say discipline names, or prize name from the text, it is better to build your own custom model.

An option to overcome this short comings of spaCy models is to create a custom model, based on the annotations of custom texts. By annotation, I mean you select a particular entity (example : Drosophilia melanogaster) in your text and label it accordingly (label: family Drosophilidae). You can then use the annotated texts as training data for the custom model. Unfortunately, manual annotation takes hours for just a couple of pages. Therefore the tool prodi.gy (https://prodi.gy/) is an added value during annotations of the custom text.

Prodi.gy uses the same inbuilt model from spacy to annotate the text, but you can correct those annotations in a web browser. You can also add labels (eg, repository, license, output). For example, Drosophilia is a ‘keyword’ or a ‘research subject’ in the text. This way you can build your own custom model.

Step 1: Buying annotation from the https://prodi.gy/

The buying proces is pretty straight forward. I choose for personal license plus some charges, total around 475 euros. My boss was happy to reimburse it from the yearly budget. You get an email with download button. Download the software to your preferred folder (Documents in my case)

Step 2: Installing prodi.gy via command Prompt from a wheel file, type:

pip install C:\Users\yourname\Documents\prodigy-1.10.6-xxxxxxxxx-win_amd64.whl

Update 15 Aug 2022: You can install the prodi.gy tool via pip and using license key at xxx below. More information on spacy https://prodi.gy/docs/install

pip install prodigy -f https://xxxx-xxxx-xxxx@download.prodi.gy

Step 3: Using spaCy pipeline to annotate text with Spacy model

With the command given below in the command prompt you get the message to copy past the link your browser to start annotation. You can annotate the text (named here: yourtext) with custom labels which are not available with the model from spacy (en_core_web_lg). Below in the command is an example of custom labels. The annotation will be saved in prodi.gy database with SQLite (named here:dataset).

The recipe with ner.correct creates data for NER by correcting the model’s suggestions. The spaCy pipeline predicts entities contained in the text, which you can remove and correct if necessary.

python -m prodigy ner.correct dataset en_core_web_lg C:\Users\yourname\Documents\yourtext.txt — label PERSON,NORP,FAC,ORG,GPE,LOC,PRODUCT,EVENT,OUTPUT,LAW,LANGUAGE,DATE,TIME,PERCENT,MONEY,QUANTITY,ORDINAL,CARDINAL,KEYWORD,GRANT,PRIZE,DISCIPLINE,EXPERTISE,TECHNIQUE,JOURNAL,REF,ACTIVITY,WEBSITE,DOI,POSITION,IMPACT,EMAIL

The recipe with ner.correct creates data for NER by correcting the model’s suggestions. The spaCy pipeline predicts entities contained in the text, which you can remove and correct if necessary.

Step 4: Initial training of the partially annotated data (named here: dataset) with component NER on word vector model in command prompt:

The recipe calls into spaCy directly and can update an existing model or train a new model from scratch.

python -m prodigy train ner dataset en_vectors_web_lg

Step 5: Generate a temporary model(tmp_model) using the dataset trained in earlier step

python -m prodigy train ner dataset en_vectors_web_lg — output C:\Users\yourname\Documents\tmp_model — eval-split 0.2 — n-iter 20

Step 6: Use temporary model to annotate further the same text (yourtext) excluding the previously annotated data named dataset

python -m prodigy ner.correct dataset_correct C:\Users\yourname\Documents\tmp_model C:\Users\yourname\Documents\yourtext.txt — label PERSON,NORP,FAC,ORG,GPE,LOC,PRODUCT,EVENT,OUTPUT,DATE,TIME,MONEY,KEYWORD,GRANT,PRIZE,DISCIPLINE,EXPERTISE,TECHNIQUE,JOURNAL,ACTIVITY,POSITION — exclude dataset

Step 7: using tmp model annotate further the next text (yournexttext) creating a new annotated dataset named dataset_correct1

python -m prodigy ner.correct dataset_correct1 C:\Users\yourname\Documents\tmp_model C:\Users\yourname\Documents\yournexttext.txt — label PERSON,FAC,ORG,GPE,LOC,PRODUCT,EVENT,OUTPUT,DATE,MONEY,KEYWORD,GRANT,PRIZE,DISCIPLINE,JOURNAL,ACTIVITY,POSITION

Step 8: Creation of new tmp model based on earlier three annotated datasets (dataset,dataset_correct,dataset_correct1)

python -m prodigy train ner dataset,dataset_correct,dataset_correct1 en_vectors_web_lg — output C:\Users\yourname\Documents\tmp_model — eval-split 0.2 — n-iter 20

Step 9: Using new tmp model annotate further the text (yournextnexttext)creating dataset_correct3 (with smaller no. of labels)

python -m prodigy ner.correct dataset_correct3 C:\Users\yourname\Documents\tmp_model C:\Users\yourname\Documents\yournextnexttext.txt — label PERSON,ORG,GPE,LOC,EVENT,OUTPUT,MONEY,KEYWORD,GRANT,PRIZE,DISCIPLINE,JOURNAL,ACTIVITY,POSITION

Step 10: creation of final model based on earlier four annotated datasets (dataset,dataset_correct,dataset_correct1, dataset_correct3). Multiple datasets will be merged and conflicts will be filtered out.

python -m prodigy train ner dataset,dataset_correct,dataset_correct1,dataset_correct3 en_vectors_web_lg — output C:\Users\yourname\Documents\tmp_model — eval-split 0.2 — n-iter 40

Step 11: To check whether more annotation will improve the model

python -m prodigy train-curve ner dataset_1,dataset_2,dataset_3,dataset_4 C:\Users\yourname\Documents\Python_scripts\tmp_model — n-iter 10

P.S. I am not paid by prodi.gy for sharing my experience with the annotation tool.
