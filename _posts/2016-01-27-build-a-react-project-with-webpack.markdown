---
layout: post
title:  "Build a React Start Project with Webpack"
date:   2016-01-27 14:09:21
author: Yang
---
I went througth the Reactjs tutorial project recently, and would like to share how I built a comment box using Reactjs.

First, if you haven't installed nodejs before on your machine, install nodejs first using command

**brew install node**

Make you project folder, inside your project folder ran command

**npm init**

Install React using commad

**npm install --save-dev react react-dom**

To compile the JSX to javascript you need to use babel, install babel using command

**npm install --save-dev babel babel-core babel-loader**

If you are using JS6 syntax, you need to install babel-preset

**npm install --save-dev babel babel-preset-react babel-preset-es2015**

And don't forget to install webpack

**npm install webpack**

Now we are going to set up our project structure

{% highlight html %}
-project root
  -config
    webpack.config.js
  -dist
    bundle.js
  -src
    -app
      components
      Comment.js
      CommentBox.js
      CommentForm.js
      CommentList.js
    -client
      -styles
        bootstrap.min.css
        main.css
      client.js
      index.html
    -server
      comments.json
      server.js
  package.json
  README.md
{% endhighlight %}

I put the webpack configuration file inside the config folder, you can also put other app configuration files inside this folder. The generated minified js file is placed under the dist folder.

Here is how I configured the webpack. The webpack will look for client.js and compile all the related js file into bundle.js.
{% highlight html %}
var path = require('path');

module.exports = {  
  entry: path.resolve(__dirname, '../src/client/client.js'),
  output: {
    path: path.resolve(__dirname, '../dist'),
    filename: 'bundle.js'
  },
  debug: true,
  devtool: "#eval-source-map",

  module: {
    loaders: [
      {
        test: /src\/.+.js$/,
        exclude: /node_modules/,
        loader: 'babel',
        query:
          {
            presets:['es2015', 'react']
          }
      }
    ]
  }
};
{% endhighlight %}

You can check the [Reactjs tutorial] [React-Tutorial] for more specified explaination of the comments box project. Here I just made the comments box source code into four components -- Comment.js, CommentBox.js, CommentForm.js, CommentList.js. The example Comment.js code is shown below, you check the complete source code of my project in [github] [Github].

{% highlight html %}
import React from 'react';
import marked from 'marked';

class Comment extends React.Component {
  rawMarkup() {
    var rawMarkup = marked(this.props.children.toString(), {sanitize: true});
    return { __html: rawMarkup };
  }

  render() {
    return (
      <div className="comment">
        <h3 className="commentAuthor">
          {this.props.author}
        </h3>
        <span className="commentContent" dangerouslySetInnerHTML={this.rawMarkup()} />
      </div>
    );
  }
}

export default Comment;
{% endhighlight %}

[React-Tutorial]: https://facebook.github.io/react/docs/tutorial.html
[Github]: https://github.com/YangLu89/Comment-Box