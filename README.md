# A simple named entity recognition custom model from scratch with annotation tool prodi.gy
Update: A new model is build with Spacy3 on 12 November 2022 for grant applications. You can use it via:

!pip install https://huggingface.co/RaThorat/en_grant/resolve/main/en_grant-any-py3-none-any.whl

#Using spacy.load().

import spacy

nlp = spacy.load(“en_grant”)

#Importing as module.

import en_grant

nlp = en_grant.load()

SpaCy can be used to extract entities from a given text. You can use some of the available natural language models from spaCy to extract information such as personsname, organisation name or location from the text. The natural language processing (NLP) named entity recognition models such as en_core_web_lg are trained on wikipedia and on a number of web pages. However, the model does not recognize custom-specific entities such as name of animal species or an academic award. However if you want to have custom labels, say discipline names, or prize name from the text, it is better to build your own custom model.

An option to overcome this short comings of spaCy models is to create a custom model, based on the annotations of custom texts. By annotation, I mean you select a particular entity (example : Drosophilia melanogaster) in your text and label it accordingly (label: family Drosophilidae). You can then use the annotated texts as training data for the custom model. Unfortunately, manual annotation takes hours for just a couple of pages. Therefore the tool prodi.gy (https://prodi.gy/) is an added value during annotations of the custom text.

Prodi.gy uses the same inbuilt model from spacy to annotate the text, but you can correct those annotations in a web browser. You can also add labels (eg, repository, license, output). For example, Drosophilia is a ‘keyword’ or a ‘research subject’ in the text. This way you can build your own custom model.

Step 1: Buying annotation software from the https://prodi.gy/
The buying process was pretty straight forward. I choose for personal license plus some charges, total around 475 euros. My boss was happy to reimburse it from the yearly budget. You get an email with download button. Download the software to your preferred folder (Documents in my case).

# Using prodigy on your laptop
Step 2: Installing prodi.gy via command Prompt from a wheel file, type:
pip install C:\Users\yourname\Documents\prodigy-1.10.6-xxxxxxxxx-win_amd64.whl

Update 15 Aug 2022: You can install the prodi.gy tool via pip and using license key at xxx below. More information on spacy https://prodi.gy/docs/install

pip install prodigy -f https://xxxx-xxxx-xxxx@download.prodi.gy

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

# Renaming the labels in NER
If you are not satisfied with the labels you gave during annotation, you can change via following process.

My annotated data contains 24 labels, which I think are a lot. I would like to reduce the number labels, for example, change labels ‘grant’ and ‘prize’ to label ‘award’. Another example would be to merge ‘event’, ‘activity’, ‘output’ ‘product’ to ‘product’.

Step: export the annotated dataset as jsonl file by using following command in the terminal

prodigy db-out dataset > ./data.jsonl

Step: install jq via the terminal with code

sudo apt-get install jq

Step: use following code to change the labels

sed 's,"label":"OLD_LABEL","label":"NEW_LABEL",g' old_data.jsonl > new_data.jsonl

You can reintegrate this new data jsonl file in the dataset of your prodigy database by command db-in:

prodigy db-in new_dataset ./new_data.jsonl --rehash

# Using prodigy in virtual machine Google cloud compute engine
My database with annotated data is 200 MB. When i tried to create a model the process got killed. I suspected it it a RAM problem as my laptop have only 4 gb RAM. Therefore Idecided to use virtual machine by one of the providers Google. On the prodigy forum I got to know of Compute engine. I have never used virtual machine before for computing.

Step setup a google cloud compute engine as shown in various youtube videos. I used Compute Engine with 64 GB RAM and ubuntu boot disk.

Step install prodigy inside the VM instance ssh. The installation of prodigy is similar to any local ubuntu laptop.
pip install prodigy -f https://xxxx-xxxx-xxxx@download.prodi.gy
Step Create sqlite3 database
Create a test.txt with some text inside it. Use the following code to create test_dataset

prodigy ner.manual test_dataset en_core_web_lg ./test.txt - label PERSON,ORG,PRODUCT
where,

ner.manual is the recipe,

test_dataset is the annotation dataset (to be created),

en_core_web_lg is the Spacy model used to annotate,

./test.txt is the path where your test.txt is for annotation and

— label PERSON,ORG,PRODUCT are the labels. You can add or change the labels as you like.

At the first run, prodigy creates prodigy.db and prodigy.json in the home folder.

# Transfer your prodigyold.db file from local place (from your local laptop) to virtual machine using ssh terminal
Step upload the prodigy.db from your local storage to virtual machine using the button ‘upload’ in ssh terminal. Move the old database file to the folder .prodigy. How to combine/merge the old database with the new database? The procedure is given below.

# How to transfer prodigyold.db (containing older prodigy annotated dataset) to new prodigy.db
Step: move the old database file to the folder .prodigy.
Check if the database name doesnot contain any spaces or symobls (otherwise you will get error in following steps). Now your .prodigy folder contains two database files (prodigy.db and prodigyold.db) and a json file (prodigy.json).

Step: change the name of the db file in the prodigy.json

{
  "db": "sqlite",
  "db_settings": {
    "sqlite": {
      "name": "prodigyold.db",
      "path": "./.prodigy"
    }
  }
}
This step can be done within ssh terminal using vi commands (more info on internet). Else, the file can be edited in the local enviornment and then uploaded in ssh terminal, and moved to .prodigy folder as told for the database above.

Step: Check which datasets are there in your old database by typing following command

prodigy stats -l

The result shows number of datasets, sessions and name of the datasets as well. There is only one dataset in my case (old_dataset). A model can be created, just by using old dataset without having to hydrate new database:

prodigy train /home/gebruiker/Documenten/ — ner old_dataset

However, it is a good practice to use those datasets to export the annotated data as JSONL files as shown below.

Step: Export the old prodigyold.db dataset using db-out command to produce a JSONL file.
Beware: before executing the db-out command, give the name of the old database (‘prodigyold.db’ in my case) in the prodigy.json file, shown above.

prodigy db-out old_dataset > ./old_data.jsonl

Step: change the name of the db file again in the prodigy.json

{
  "db": "sqlite",
  "db_settings": {
    "sqlite": {
      "name": "prodigy.db",
      "path": "./.prodigy"
    }
  }
}
Step: create new_dataset in the prodigy.db using the annotated old_data.jsonl
prodigy db-in new_dataset ./old_data.jsonl --rehash
You can use the new_dataset to create a model using prodigy train recipe.

# Previous version of this article with detailed building of models with Spacy 2
# Note: the model was built last year June 2021 with Spacy 2.3.7. The codes are no longer up to date (august 2022). However this article gives a general idea how to build a model with prodi.gy.

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
