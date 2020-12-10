# Architecture


## Introduction

In the following the main idea of this software is explained. Afterwards a short introduction of the software architecture follows including examples how to extend the software with new features.
The currently main goal is that this software works as a service to find semantically similar COLID resources for a given COLID resource or some partial information (titel, description).

To archive this, Natural Language Processing is used. In short, we use the python package `spacy` to use pretrained nlp models (like BERT) and we use `gensim`
to train own `word2vec` and `fasttext` models. This different models are used for the same porpuse. To calulate a similarity between two ressources we compare the similarity of each free-text-field COLID resources have, which are titel, description and editorial note.
Now, to calculate a similarity between two free-text-fields we use the vector-representation, also called word embeddings for each word, using nlp models.

To calculate the similarity between two word embedding vectors we use the cosine function. A value from 0.6 to 1 indicates high similarity, where a value less or equal 0.25 means no relationat all.

To speed up the comparing process the embedding vectors of existing resources needs to  precalulated. Also, a nlp model needs to learn word embeddings based on a corpus. The corpus of COLID changes which required new training of a model. To keep the service up-to-date background jobs are running asynchronously in the background while the service is available to preprocess needed data and models continuously. The running API service also can the newest model in production.

## Architecture

The idea is to build a software architecture based on the devon (https://devonfw.com/website/pages/welcome/welcome.html) framework, but for a data-driven platform.
We therefore extend the `data`-layer to have input and output/result entities which handle the different data sources. The `logic` layer also contains a nested package `business` where use-case seperated *data-processing* logic is implemented.
The `service` layer is also used, where the `WebService` is implemented and `BackgroundJobs` are defined. 

As this is a data driven application we more want to pipe data transformation through several calculations. This is implemented by having 
a ``DataPipeline`` class, which accepts input and output instances and a `BaseTransform` which process data from several sources. 

The *"Inputs"* and *"Outputs"* in such a `DataPipeline` are instances of `BaseInput`, resp. `BaseResult`. The idea is simple. This objects provide a `get` resp. `save` method
where all the logic of laoding and saving is encapsulated in the respective classes.

Is a ```DataPipeline``` defined, it can be used to run some "User Story", i.e. some defined processing steps.

A ``DataPipeline`` is often used in a ``Processing`` class, where inputs from a service (i.e. REST API) are processed to easily run the pipeline.

To make this application complete, configurations and secret keys are stored in the `common` layer. There is the class `AppConfig` which just contains all constants and parameter to run the application successfully.
The `AppSetup` file use the config class and make it public available, which means, that in the application every module can import the `global_app_setup` object.

See also the example section.

# Examples

## Simple DataPipeline Example

````python

    from dataaccess.results.printresult import PrintResult
    from dataaccess.results.simplevarresult import SimpleVarResult
    from dataaccess.inputs.pickle import PickleInput
    from dataaccess.inputs.simplevarinput import SimpleVarInput
    from logic.business.similarity.transformation.Str2NLPDocTransform import Str2NLPDocTransform
    from logic.tools.datapipeline.DataPipeline import DataPipeline
    from logic.business.similarity.transformation.NLPDoc2SIFEmbeddingTransform import NLPDoc2SIFEmbeddingTransform
    from pprint import pprint

    # a DataPipeline instance
    pipeline = DataPipeline()

    # a pipeline reads in data with the `read` method.
    # This source must have a name to access it later on
    # The method requires and object of Type 'InputBase',
    # in this case a variable can be read in
    pipeline.read(SimpleVarInput("An input sentence a user has submitted"), "text")

    # The interesting part are the transformer. In fact these are simple functions
    # which runs some processing of a given input. It makes much sense to make transformers not so small,
    # put a logic entity in it. Here we convert a string to a word embedding, these means:
    # 1) process the input / tokenization
    # 2) calculate embedding for each word
    # The syntax here means, a transformer requires slots, where data it can read data from or write data to
    # The names are like "columns" of a dataframe. But it is not a dataframe, more a dataset
    pipeline.transform(Str2NLPDocTransform().input_slot("text").output_slot("doc"))

    # Everything can chained together, like several transformers or inputs, however we now transform the doc object to
    # a SIF embedding (weighted average). This transformer need some other calculation, which can be saved in some
    # S3 storage or in this case in a pickle-file
    pipeline.read(PickleInput("src_colid_word_count"), "wordcount")

    # Now the transformation can be added
    pipeline.transform(NLPDoc2SIFEmbeddingTransform().input_slot("wordcount", "doc").output_slot("embed"))

    # Finally we need to access the generated content. The simples way is to print it. The `write` method
    # need an object of type "BaseResult". This object needs to now which sources should be saved, so the 'set_slots'
    # method need to be called
    #
    # One can write his own ResultEntity
    pipeline.write(PrintResult().set_slots("embed"))

    # The next simples possibility is to write it into a variable
    # The "normal case" would be to write output to some Database or Storage or something like that
    # Don't forget the `set_slots` method
    output_embedding = SimpleVarResult()
    pipeline.write(output_embedding.set_slots("embed"))

    # Until now, nothing has happend, we start the pipeline with `run`:
    pipeline.run()

    # has everything process as assumed we can now also access the output
    pprint(output_embedding)

````


## Simple Processing Example - equivalent to logic implementations (UC)

See https://devonfw.com/website/pages/docs/devon4j.asciidoc_coding-conventions.html for more details


````python

# Create a new class which wrappes a pipeline to a callable object.
# The Process is in the best case a user story which is implemented here with a pipeline.
# In this example we want to count each word of an input text
class SimpleProcess(BaseProcess):
    def pipeline(self) -> DataPipeline:
        # the `inputs` field defines the arguments of the callable object, which can be none or many
        self.inputs = ["text"]
        # the output field define which slots should be returned
        self.output = SimpleVarResult().add_slot("counts")

        # here we define a Data Pipeline
        pipeline = DataPipeline()
        # If an input argument is define, we need to add it to the pipeline,
        # then the BaseProcess injects it correctly to the pipeline
        pipeline.read(SimpleVarInput(), "text")
        # The NLPDocs2WordCountsTransform transformer needs an empty counter (even None is allowed)
        pipeline.read(SimpleVarInput(Counter()), "word_count")
        # transform a text to word embedding / a NLP-doc representation
        pipeline.transform(Str2NLPDocTransform().input_slot("text").output_slot("doc"))
        # transform a doc representation to word count
        pipeline.transform(NLPDocs2WordCountsTransform().input_slot("word_count", "doc").output_slot("counts"))
        # we need to write the result to our output, which then is returned in the call
        pipeline.write(self.output)
        # we are done :)
        return pipeline


# Use this defined process as follows (in a webservice, in a script)
process = SimpleProcess()

# call it
process("A simple text as small as possible to get a useful experience of the simple process.")

````

## Write own Input Object

*Note: The structure of dataaccess implementation is as follows. In the main packages `input` and `result` does exist a packge for every class, where inside this package the api for the user and the concrete implementation are seperated*

### File-Structure

We want to create a Input-Layer which loads weather data from an public API. We name this class `WeatherAPIInput`, so the package is 
called ``weather-api`` where inside there is an `api` and an `impl` package. Inside the `api` package we create the `WeatherAPIInput` class.

The complete structure for this layer is:

````bash

inputs/
    weather-api/
        api/
            WeatherAPIInput.py
        impl/
            <empty>
````

### Code-Structure

````python

import requests
# This weather data input class needs to overwrite the `get` and `types` method. This is exmaple is not very complex, but whatever has to be done can
# be outsourced in some other method or in the `impl` package. The `get` method takes no parameter, so a configuration can be done within
# the constructor
class WeatherAPIInput(BaseInput):
    def __init__(self, city="London,uk"):
        self.city = city
        # The default construcor should be called, too
        super(WeatherAPIInput, self).__init__()

    # This get method will be called in the Pipeline
    def get(self):
        return self.__perform_access()

    # The `types` method is only used to see which data types this class returns, this feature is not used often,
    # however this methods needs to be overwritten
    def types(self):
        return dict

    # Some useful implementation where data are loaded from
    # in this case we use a parameter, which can be extended of course
    def __perform_access(self):
        url = f'http://api.openweathermap.org/data/2.5/weather?q={self.city}&APPID=df704cccc37249d0375d4da84570ee9a'
        response = requests.get(url)
        if response.status_code == 200:
            return response.json()
        else:
            return dict()

# Example call if one wants to use the weather input layer standalone:
WeatherAPIInput("Frankfurt,de").get()

````


## Write own Result Object

*Note: The structure of the result layer follows the guidelines of the input layer*

### File-Structure

For the result layer, I just do not have any nice idea, so I explain the simple `PickleResult` class. 
The package calls ``pickle`` . The complete structure for this layer is:

````bash

results/
    pickle/
        api/
            PickleResult.py
        impl/
            <empty>
````

### Code-Structure of a Result entity


````python

import pickle
from os.path import dirname
from dataaccess.results.base import BaseResult

# Extending the class BaseResults, is a simple task, we only need to extend the 'save' method. In the constructor we 
# can define some configuration, like here a name for the pickle file
class PickleResult(BaseResult):
    def __init__(self, filename):
        super(PickleResult, self).__init__()
        self.filename = filename
    
    # the logic is not a very complex thing, for this project I used it to store some variable for reusing in one 
    # storage folder.
    # Important to know is, that the save method has several arguments, saved in *args. It is needed to think about what exactly need to save 
    # and maybe raise an Excpetion if multiple inputs makes no sense  
    def save(self, *args):
        file = dirname(__file__) + "/../../../../storage/" + self.filename
        with open(file, "wb") as f:
            if len(args) == 1:
                return pickle.dump(args[0], f)
            else:
                return pickle.dump(args, f)

````

## Write own Transformer 

We now want to write a transformer which takes our weather data and extract some usefull information (this is a fun example) 

````python

# To get a custom transformer run, we simply need to implement two methods.
# The core is of course the transform method, as many input types we defined as many argument it needs 
class Weather2HumanTransform(BaseTransform):
    def transform(self, data):
        temp_celsius = data["main"]["feels_like"]-273.15
        forecast = f'We present you current weather forecast for the city of {data["name"]} ({data["coord"]["lon"]},{data["coord"]["lat"]}), where the wind has a speed of {data["wind"]["speed"]} and the humidity is {data["main"]["humidity"]} where it feels like {temp_celsius}°C.'        
        return forecast
````
