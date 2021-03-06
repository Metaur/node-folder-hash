﻿Create a hash checksum over a folder or a file.  
The hashes are propagated upwards, the hash that is returned for a folder is generated over all the hashes of its children.  
The hashes are generated with the _sha1_ algorithm and returned in _base64_ encoding by default.

Each file returns a name and a hash, and each folder returns additionally an array of children (file or folder elements).  

## Usage 
First, install folder-hash with `npm install --save folder-hash` or `yarn add folder-hash`.  

### Simple example
See file *./examples/readme-example1.js*.  
This example excludes all files and folders starting with a dot, (e.g. *.git/* and *.gitignore*), the *node_modules* folder.  

```js
const { hashElement } = require('folder-hash');

const options = {
    folders: { exclude: ['.*', 'node_modules', 'test_coverage'] },
    files: { include: ['*.js', '*.json'] }
};

console.log('Creating a hash over the current folder:');
hashElement('.', options)
    .then(hash => {
        console.log(hash.toString());
    })
    .catch(error => {
        return console.error('hashing failed:', error);
    });
```

The returned information looks for example like this:
```
Creating a hash over the current folder:
{ name: '.', hash: 'YZOrKDx9LCLd8X39PoFTflXGpRU=,'
  children: [
    { name: 'examples', hash: 'aG8wg8np5SGddTnw1ex74PC9EnM=,'
      children: [
        { name: 'readme-example1.js', hash: 'Xlw8S2iomJWbxOJmmDBnKcauyQ8=' }
        { name: 'readme-with-callbacks.js', hash: 'ybvTHLCQBvWHeKZtGYZK7+6VPUw=' }
        { name: 'readme-with-promises.js', hash: '43i9tE0kSFyJYd9J2O0nkKC+tmI=' }
        { name: 'sample.js', hash: 'PRTD9nsZw3l73O/w5B2FH2qniFk=' }
      ]}
    { name: 'index.js', hash: 'kQQWXdgKuGfBf7ND3rxjThTLVNA=' }
    { name: 'package.json', hash: 'w7F0S11l6VefDknvmIy8jmKx+Ng=' }
    { name: 'test', hash: 'H5x0JDoV7dEGxI65e8IsencDZ1A=,'
      children: [
        { name: 'parameters.js', hash: '3gCEobqzHGzQiHmCDe5yX8weq7M=' }
        { name: 'test.js', hash: 'kg7p8lbaVf1CPtWLAIvkHkdu1oo=' }
      ]}
  ]}
```


It is also possible to only match the full path and not the basename. The same configuration could look like this:  
_But unfortunately *nix and Windows behave differently, so please use caution._
```js
const options = {
    folders: {
        exclude: ['.*', '**.*', '**node_modules', '**test_coverage'],
        matchBasename: false, matchPath: true
    },
    files: {
        //include: ['**.js', '**.json' ], // Windows
        include: ['*.js', '**/*.js', '*.json', '**/*.json'], // *nix
        matchBasename: false, matchPath: true
    }
};
```


### Other examples using promises
See file *./examples/readme-with-promises.js*
```js
const path = require('path');
const { hashElement } = require('folder-hash');

// pass element name and folder path separately
hashElement('test', path.join(__dirname, '..'))
  .then(hash => {
    console.log('Result for folder "../test":', hash.toString(), '\n');
  })
  .catch(error => {
    return console.error('hashing failed:', error);
  });

// pass element path directly
hashElement(__dirname)
  .then(hash => {
    console.log(`Result for folder "${__dirname}":`);
    console.log(hash.toString(), '\n');
  })
  .catch(error => {
    return console.error('hashing failed:', error);
  });

// pass options (example: exclude dotFolders)
const options = { encoding: 'hex', folders: { exclude: ['.*'] } };
hashElement(__dirname, options)
  .then(hash => {
    console.log('Result for folder "' + __dirname + '" (with options):');
    console.log(hash.toString(), '\n');
  })
  .catch(error => {
    return console.error('hashing failed:', error);
  });
```

### Other examples using error-first callbacks
See *./examples/readme-with-callbacks.js*

```js
const path = require('path');
const { hashElement } = require('folder-hash');

// pass element name and folder path separately
hashElement('test', path.join(__dirname, '..'), (error, hash) => {
    if (error) {
        return console.error('hashing failed:', error);
    } else {
        console.log('Result for folder "../test":', hash.toString(), '\n');
    }
});

// pass element path directly
hashElement(__dirname, (error, hash) => {
    if (error) {
        return console.error('hashing failed:', error);
    } else {
        console.log('Result for folder "' + __dirname + '":');
        console.log(hash.toString(), '\n');
    }
});

// pass options (example: exclude dotFiles)
const options = { algo: 'md5', files: { exclude: ['.*'], matchBasename: true } };
hashElement(__dirname, options, (error, hash) => {
    if (error) {
        return console.error('hashing failed:', error);
    } else {
        console.log('Result for folder "' + __dirname + '":');
        console.log(hash.toString());
    }
});
```

### Parameters for the hashElement function

<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Attributes</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>name</td>
            <td>
                <span>string</span>
            </td>
            <td>
            </td>
            <td>element name or an element's path</td>
        </tr>
        <tr>
            <td>dir</td>
            <td>
                <span>string</span>
            </td>
            <td>
                &lt;optional&gt;<br>
            </td>
            <td>directory that contains the element (generated from name if omitted)</td>
        </tr>
        <tr>
            <td>options</td>
            <td>
                <span>Object</span>
            </td>
            <td>
                &lt;optional&gt;<br>
            </td>
            <td>
                Options object (see below)
            </td>
        </tr>
        <tr>
            <td>callback</td>
            <td>
                <span>fn</span>
            </td>
            <td>
                &lt;optional&gt;<br>
            </td>
            <td>Error-first callback function</td>
        </tr>
    </tbody>
</table>

#### Options object properties
##### Default values
```js
{
    algo: 'sha1',       // see crypto.getHashes() for options
    encoding: 'base64', // 'base64', 'hex' or 'binary'
    files: {
        exclude: [],
        include: [],
        matchBasename: true,
        matchPath: false,
        ignoreRootName: false
    },
    folders: {
        exclude: [],
        include: [],
        matchBasename: true,
        matchPath: false,
        ignoreRootName: false
    }
}
```

<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Attributes</th>
            <th>Default</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>algo</td>
            <td>
                <span>string</span>
            </td>
            <td>
                &lt;optional&gt;<br>
            </td>
            <td>
                'sha1'
            </td>
            <td>checksum algorithm, see options in crypto.getHashes()</td>
        </tr>
        <tr>
            <td>encoding</td>
            <td>
                <span>string</span>
            </td>
            <td>
                &lt;optional&gt;<br>
            </td>
            <td>
                'base64'
            </td>
            <td>encoding of the resulting hash. One of 'base64', 'hex' or 'binary'</td>
        </tr>
        <tr>
            <td>files</td>
            <td>
                <span>Object</span>
            </td>
            <td>
                &lt;optional&gt;<br>
            </td>
            <td colspan="2">
                Rules object (see below)
            </td>
        </tr>
        <tr>
            <td>folders</td>
            <td>
                <span>Object</span>
            </td>
            <td>
                &lt;optional&gt;<br>
            </td>
            <td colspan="2">
                Rules object (see below)
            </td>
        </tr>
    </tbody>
</table>

#### Rules object properties
<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Type</th>
            <th>Attributes</th>
            <th>Default</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>exclude</td>
            <td>
                <span>Array.&lt;string&gt;</span>
            </td>
            <td>
                &lt;optional&gt;<br>
            </td>
            <td>
                []
            </td>
            <td>Array of optional exclude glob patterns, see <a href="https://github.com/isaacs/minimatch#features">minimatch doc</a></td>
        </tr>
        <tr>
            <td>include</td>
            <td>
                <span>Array.&lt;string&gt;</span>
            </td>
            <td>
                &lt;optional&gt;<br>
            </td>
            <td>
                []
            </td>
            <td>Array of optional include glob patterns, see <a href="https://github.com/isaacs/minimatch#features">minimatch doc</a></td>
        </tr>
        <tr>
            <td>matchBasename</td>
            <td>
                <span>bool</span>
            </td>
            <td>
                &lt;optional&gt;<br>
            </td>
            <td>
                true
            </td>
            <td>Match the glob patterns to the file/folder name</td>
        </tr>
        <tr>
            <td>matchPath</td>
            <td>
                <span>bool</span>
            </td>
            <td>
                &lt;optional&gt;<br>
            </td>
            <td>
                false
            </td>
            <td>Match the glob patterns to the file/folder path</td>
        </tr>
        <tr>
            <td>ignoreRootName</td>
            <td>
                <span>bool</span>
            </td>
            <td>
                &lt;optional&gt;<br>
            </td>
            <td>
                false
            </td>
            <td>Set to true to calculate the hash without the basename of the root (first) element</td>
        </tr>
    </tbody>
</table>


## Behavior
The behavior is documented and verified in the unit tests. Execute `npm test` or `mocha test`, and have a look at the _test_ subfolder.  
You can also have a look at the [CircleCI report. ![CircleCI](https://circleci.com/gh/marc136/node-folder-hash/tree/master.svg?style=svg)](https://circleci.com/gh/marc136/node-folder-hash/tree/master)


### Creating hashes over files
**The hashes are the same if:**

- A file is checked again
- Two files have the same name and content (but exist in different folders)

**The hashes are different if:**

- A file was renamed or its content was changed
- Two files have the same name but different content
- Two files have the same content but different names

### Creating hashes over folders
Content means in this case a folder's children - both the files and the subfolders with their children.

**The hashes are the same if:**

- A folder is checked again
- Two folders have the same name and content (but have different parent folders)

**The hashes are different if:**

- A file somewhere in the directory structure was renamed or its content was changed
- Two folders have the same name but different content
- Two folders have the same content but different names

## License
MIT, see LICENSE.txt
