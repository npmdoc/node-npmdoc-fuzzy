# api documentation for  [fuzzy (v0.1.3)](https://github.com/mattyork/fuzzy)  [![npm package](https://img.shields.io/npm/v/npmdoc-fuzzy.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-fuzzy) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-fuzzy.svg)](https://travis-ci.org/npmdoc/node-npmdoc-fuzzy)
#### small, standalone fuzzy search / fuzzy filter. browser or node

[![NPM](https://nodei.co/npm/fuzzy.png?downloads=true)](https://www.npmjs.com/package/fuzzy)

[![apidoc](https://npmdoc.github.io/node-npmdoc-fuzzy/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-fuzzy_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-fuzzy/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-fuzzy/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-fuzzy/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Matt York",
        "email": "york.matt@gmail.com",
        "url": "mattyork.org"
    },
    "bugs": {
        "url": "https://github.com/mattyork/fuzzy/issues"
    },
    "dependencies": {},
    "description": "small, standalone fuzzy search / fuzzy filter. browser or node",
    "devDependencies": {
        "chai": ">= 1.1.1",
        "jshint": ">= 0.7.1",
        "mocha": ">= 1.3.0",
        "uglify-js": ">= 1.3.2",
        "underscore": ">= 1.3.3"
    },
    "directories": {},
    "dist": {
        "shasum": "4c76ec2ff0ac1a36a9dccf9a00df8623078d4ed8",
        "tarball": "https://registry.npmjs.org/fuzzy/-/fuzzy-0.1.3.tgz"
    },
    "engines": {
        "node": ">= 0.6.0"
    },
    "gitHead": "39e3f256ce44411bc20ee79bc6bbf616ac88d163",
    "homepage": "https://github.com/mattyork/fuzzy",
    "keywords": [
        "fuzzy",
        "search",
        "filter",
        "sublime",
        "sublime text"
    ],
    "licenses": [
        {
            "type": "MIT",
            "url": "https://github.com/mattyork/fuzzy/blob/master/LICENSE-MIT"
        }
    ],
    "main": "lib/fuzzy",
    "maintainers": [
        {
            "name": "mattyork",
            "email": "york.matt@gmail.com"
        }
    ],
    "name": "fuzzy",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git://github.com/mattyork/fuzzy.git"
    },
    "scripts": {
        "test": "mocha"
    },
    "typings": "lib/fuzzy",
    "version": "0.1.3"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module fuzzy](#apidoc.module.fuzzy)
1.  [function <span class="apidocSignatureSpan">fuzzy.</span>filter (pattern, arr, opts)](#apidoc.element.fuzzy.filter)
1.  [function <span class="apidocSignatureSpan">fuzzy.</span>match (pattern, str, opts)](#apidoc.element.fuzzy.match)
1.  [function <span class="apidocSignatureSpan">fuzzy.</span>simpleFilter (pattern, array)](#apidoc.element.fuzzy.simpleFilter)
1.  [function <span class="apidocSignatureSpan">fuzzy.</span>test (pattern, str)](#apidoc.element.fuzzy.test)



# <a name="apidoc.module.fuzzy"></a>[module fuzzy](#apidoc.module.fuzzy)

#### <a name="apidoc.element.fuzzy.filter"></a>[function <span class="apidocSignatureSpan">fuzzy.</span>filter (pattern, arr, opts)](#apidoc.element.fuzzy.filter)
- description and source-code
```javascript
filter = function (pattern, arr, opts) {
  if(!arr || arr.length === 0) {
    return [];
  }
  if (typeof pattern !== 'string') {
    return arr;
  }
  opts = opts || {};
  return arr
    .reduce(function(prev, element, idx, arr) {
      var str = element;
      if(opts.extract) {
        str = opts.extract(element);
      }
      var rendered = fuzzy.match(pattern, str, opts);
      if(rendered != null) {
        prev[prev.length] = {
            string: rendered.rendered
          , score: rendered.score
          , index: idx
          , original: element
        };
      }
      return prev;
    }, [])

    // Sort by score. Browsers are inconsistent wrt stable/unstable
    // sorting, so force stable by using the index in the case of tie.
    // See http://ofb.net/~sethml/is-sort-stable.html
    .sort(function(a,b) {
      var compare = b.score - a.score;
      if(compare) return compare;
      return a.index - b.index;
    });
}
```
- example usage
```shell
...

## Use it

Padawan: Simply filter an array of strings.

'''javascript
var list = ['baconing', 'narwhal', 'a mighty bear canoe'];
var results = fuzzy.filter('bcn', list)
var matches = results.map(function(el) { return el.string; });
console.log(matches);
// [ 'baconing', 'a mighty bear canoe' ]
'''

Jedi: Wrap matching characters in each string
...
```

#### <a name="apidoc.element.fuzzy.match"></a>[function <span class="apidocSignatureSpan">fuzzy.</span>match (pattern, str, opts)](#apidoc.element.fuzzy.match)
- description and source-code
```javascript
match = function (pattern, str, opts) {
  opts = opts || {};
  var patternIdx = 0
    , result = []
    , len = str.length
    , totalScore = 0
    , currScore = 0
    // prefix
    , pre = opts.pre || ''
    // suffix
    , post = opts.post || ''
    // String to compare against. This might be a lowercase version of the
    // raw string
    , compareString =  opts.caseSensitive && str || str.toLowerCase()
    , ch;

  pattern = opts.caseSensitive && pattern || pattern.toLowerCase();

  // For each character in the string, either add it to the result
  // or wrap in template if it's the next string in the pattern
  for(var idx = 0; idx < len; idx++) {
    ch = str[idx];
    if(compareString[idx] === pattern[patternIdx]) {
      ch = pre + ch + post;
      patternIdx += 1;

      // consecutive characters should increase the score more than linearly
      currScore += 1 + currScore;
    } else {
      currScore = 0;
    }
    totalScore += currScore;
    result[result.length] = ch;
  }

  // return rendered string if we have a match for every char
  if(patternIdx === pattern.length) {
    // if the string is an exact match with pattern, totalScore should be maxed
    totalScore = (compareString === pattern) ? Infinity : totalScore;
    return {rendered: result.join(''), score: totalScore};
  }

  return null;
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.fuzzy.simpleFilter"></a>[function <span class="apidocSignatureSpan">fuzzy.</span>simpleFilter (pattern, array)](#apidoc.element.fuzzy.simpleFilter)
- description and source-code
```javascript
simpleFilter = function (pattern, array) {
  return array.filter(function(str) {
    return fuzzy.test(pattern, str);
  });
}
```
- example usage
```shell
n/a
```

#### <a name="apidoc.element.fuzzy.test"></a>[function <span class="apidocSignatureSpan">fuzzy.</span>test (pattern, str)](#apidoc.element.fuzzy.test)
- description and source-code
```javascript
test = function (pattern, str) {
  return fuzzy.match(pattern, str) !== null;
}
```
- example usage
```shell
n/a
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
