# ngram-word-generator

Word generation based on n-gram models, and a cli utility to generate said models.

N-gram models (see on [wikipedia](https://en.wikipedia.org/wiki/N-gram)) are often used among other things to generate sentences and can produce satisfying results, though the inherent lack of context tracking and actual grammar checking is a severe shortcoming.
When used as a word generator this simple algorithm really shines at producing new, imaginary words. For example here are few names generated based on French firstnames - *Mélaïde, Hadrigue, Laudence, Vestine, Pasien, Eulalisse* - and on Irish firstnames - *Ceallán, Muiríne, Toirdre, Finnbhach, Dáirtín, Éannailís*.

A previous version of the project can be seen in use in [Stochastèmes](http://www.kchapelier.com/stochastemes/), a word generator based on poetic corpora.

## Features

 - Works in node and in browsers
 - A cli utility to do the heavy-lifting of generating n-gram models from text files
 - The generated models are standard json files, optimized for web transfer
 - Porting the client code to other languages should be trivial, the generated n-gram models can thus be used virtually anywhere

## Installing and testing

With [npm](http://npmjs.org) do:

```
npm install ngram-word-generator
```

It can also be installed globally to simplify the access to the cli utility:

```
npm install ngram-word-generator -g
```

## Basic example

```js
var makeGenerator = require('ngram-word-generator'),
    ngramModel = require('./irish-firstnames.json'); //generated with the cli utility

var generator = makeGenerator(ngramModel);

console.log(generator(10));
```

## Public API

### makeGenerator(ngramModel)

Create a generator based on a given n-gram model.

**Options**

 - *ngramModel :* An n-gram model as generated by the cli utility.

### generator(lengthHint, rngFunction)

Generate a word based on the n-gram model.

**Options**

 - *lengthHint :* Suggest the desired word length. The nature of the algorithm makes it impossible to ensure the exact length of the result (except through brute-force).
 - *rngFunction :* Random number generator function. Default to Math.random.

### generator.dictionary(lengthMin, lengthMax)

Generate a comprehensive dictionary (list) of all the generated words included in the given length range.

**Options**

 - *lengthMin :* Minimum length of the words included in the dictionary.
 - *lengthMax :* Maximum length of the words included in the dictionary. Default to lengthMin.

## CLI

The cli utility is used to generate the n-gram models

`ngram-word-generator source.txt > model.json`

In Unix-like environment, the cli utility can also be piped into :

`cat source.txt | ngram-word-generator > model.json`

### ngram-word-generator sourceFile [options]

*sourceFile* must be an utf8 text file. No specific structure is required.

**Options**

 - name: A name for the model, not directly used.
 - n: The order of the model (1: unigram, 2: bigram, 3: trigram, etc.). Default to 3.
 - minLength: The minimum length of the word considered in the generation of the model. Default to 4.
 - unique: Usually if multiple instances of a specific word is included in the source file, the model will be skewed toward generating similar words. Setting this option ensure this doesn't happen.
 - compress: Reduce the size of the model file, making it less readable and slightly less accurate.
 - excludeOriginal: The model will include the full list of the words included in the source file so that the generation can blacklist them.
 - filter: Character filtering option, either one the filters listed below or a regex. Default to 'extended'.

**Filters**

 - none: All non-whitespace characters
 - alphabetical: A to Z
 - numerical: 0 to 9
 - alphaNumerical: A to Z and 0 to 9
 - extended: A to Z and éèëêęėēúüûùūçàáäâæãåāíïìîįīóöôòõœøōñńß
 - extendedNumerical: A to Z, 0 to 9 and éèëêęėēúüûùūçàáäâæãåāíïìîįīóöôòõœøōñńß
 - french: A to Z and éèëêúüûùœàáäâæíïìîóöôòç
 - english: A to Z and œæ
 - oldEnglish: A to Z and þðƿȝæœ
 - chinese: All the Chinese characters
 - japanese: All the Japanese (and Chinese) characters
 - noSymbols: Exclude all symbols and punctuation from Latin unicode blocks and specific punctuation, symbols and dingbats unicode blocks.

**Full example**

```
ngram-word-generator source.txt --n=3 --minLength=4 --filter=noSymbols --compress --unique --excludeOriginal > model.json
```

With a custom regular expressions as filter (all characters out of the A-U range will be filtered out) :

```
ngram-word-generator source.txt --n=3 --minLength=4 --filter=/[^a-u]+/ig --compress --unique --excludeOriginal > model.json
```

## Model generation without the CLI utility

The model generation being a costly process, it is recommended to use the cli utility described above.

```js
var generateModel = require('ngram-word-generator/model-generation'); // specific entry point

var textData = ...; // retrieve a large text as a single string somehow

var ngramModel = generateModel(textData, {
    name: 'My n-gram model',
    filter: 'noSymbols',
    n: 3,
    minLength: 4,
    unique: false,
    excludeOriginal: true,
    compress: true
});

console.log(ngramModel);

// generate a word with the model

var makeGenerator = require('ngram-word-generator');

var generator = makeGenerator(ngramModel);

console.log(generator(10));
```

### generateModel(textData, options)

Generate an n-gram model based on a given text.

 - *textData:* Text corpus as a single, preferably large, string.
 - *options.name:* Name of the n-gram model, not directly used.
 - *options.n:* Order of the model (1: unigram, 2: bigram, 3: trigram, ...). Default to 3.
 - *options.minLength:* Minimum length of the word considered in the generation of the model. Default to options.n. Must be larger than or equal to options.n, an error will be thrown otherwise.
 - *options.unique:* Avoid skewing the generation toward the most repeated words in the text corpus. Default to false.
 - *options.compress:* Reduce the size of the model file, making it slightly less accurate. Default to false.
 - *options.excludeOriginal:* Include the full list of the words considered in the generation so they can be blacklisted. Default to false.
 - *options.filter:* Character filtering option, either one the existing filters (see CLI) or a RegExp object. Default to 'extended'.

## Changemap

### [1.2.0](https://github.com/kchapelier/ngram-word-generator/tree/1.2.0) (2016-09-01) :

 * Allow and document how to generate n-gram models without the cli utility.

### [1.1.0](https://github.com/kchapelier/ngram-word-generator/tree/1.1.0) (2016-08-19) :

 * Add built-in filters (french, english, oldEnglish, chinese, japanese and noSymbols).
 * Allow a custom regular expression as filter for the cli utility.
 * Add a dictionary method to generate a comprehensive list of words.

### [1.0.0](https://github.com/kchapelier/ngram-word-generator/tree/1.0.0) (2016-08-12) :

 * First publication.

## Roadmap

 - Fix issue where a model with exclude could lead to an infinite loop with a text corpus of poor quality
 - Make an online tool to generate the n-gram models

## License

MIT
