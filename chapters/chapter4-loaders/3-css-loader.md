## Loading CSS Files

**Note**: From this point on, I assume that you have your work folder, main file, and Webpack config file are setup. For more information, see the previous section.

In order to load CSS files, you need two loaders, `style` and `css`. Let's go ahead and install them:

```shell
npm i css-loader style-loader
```

After that, we need to add the loader definition in the loaders array. After you add the loader, your `webpack.config.js` file should look like this:

```javascript
var path = require('path');
module.exports = {
  entry: './main.js',
  output: {
    path: path.resolve('./dist'),
    filename: 'bundle.js'
  },

  module: {
    loaders: [
      {
        test: /\.html$/,
        loader: 'raw'
      },
      // ------ new stuff ------
      {
        test: /\.css$/,
        loader: 'style!css' // passing through two loaders:
                            // css-loader -> style-loader
                            // Note the order is from right to left
                            // not left to right.
                            // The `!` acts like a pipe.
      }
      // ------ new stuff ------
    ]
  }
};
```

- As you can notice, we have added a loader definition for css files. Here we are telling Webpack to run any file that ends with a `.css` extension pass it through two loaders: The CSS loader first and then the style loader.

- If you read `loader: style!css` from left to right, you might think that the css file is passed through the `style` loader first, and then the `css` loader. However, that's not the case, Webpack reads it from right to left where `!` acts like a pipe.

Now, if you run `webpack` you should get the output in the `bundle.js` file. If you look at the output file, you can see this line where the content of the file is exported:


```javascript
exports.push([module.id, "body {\n  background-color: gray;\n}\n", ""]);
```

As you can see the content of the css file has been exported as a string literal. But the magic happens with the `style` loader and the `css` loader. First, the `style` loader kicks in and adds some css to the DOM by adding a style tag. Then the `css` loader kicks in and does the heavy lifting. By default, the styles are added to the bottom of the head section on the page. By adding a style tag on the page, no http request is made to actually load the css file. If you need to output actual css files, Webpack can help you with that too. In the next section, we are going to talk just about that.

### Creating Separate CSS Outputs

In order to create a separate css file instead, we need to use the `extract-text-webpack-plugin` plugin. We haven't talked about plugins yet, but for now we don't have worry so much about it. All we need to know is that we can use this particular plugin to spit out a separate CSS file. Update your `webpack.config.js` file to look like the following:

```javascript
var path = require('path');
// ------------ Require the Plugin ------------ \\
var ExtractTextPlugin = require('extract-text-webpack-plugin');
// ------------ Require Webpack ------------ \\
var webpack = require('webpack');

module.exports = {
  entry: './main.js',
  output: {
    path: path.resolve('./dist'),
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      {
        test: /\.html$/,
        loader: 'raw'
      },
      {
        test: /\.css$/,
        // ------ Use the plugin to extract the content ------ \\
        loader: ExtractTextPlugin.extract('style-loader', 'css-loader')
      }
    ],
  },
  // ------ Register the plugin with Webpack ------
  plugins: [
    new ExtractTextPlugin('main.css') // <- name the output file: main.css
                                      // The result would be placed in `dist`
  ]
};
```

Notice that we are requiring the plugin, so make sure to install that with `npm i extract-text-webpack-plugin -D`. After you installed the plugin, you can then use in the loader config section to extract the result of `style!css`. Also note that we added an additional field called `plugins` and we have initialized the plugin with the name of the final output. Now if you run `webpack` you should see a file create in `dist/main.css`

