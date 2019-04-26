---
layout: post
title: graphs and flowcharts with mermaid & react & docker
date: 2019-02-24 12:00:00 +0100
categories: [javascript]
tags: [mermaid, javascript, react, docker, flowchart]
published: true
---

🧜‍♀️ If you use DDT (data-driven testing) methodology for UI testing, you might have come across a similar problem as I did. I had to create the **graphic representation of user flow** in the application, I'm testing. Like some kind of graphs or flowcharts that could be used both by automation and manual testing teams. We would use them as a wireframe for test scenarios and test cases.

Flowcharts that I was about to create had to meet certain expectations:
1. built into the automation testing framework
2. utilize the same data I'm using for automated tests
3. automatically created
4. easy to maintain
5. available for everybody to see them at any time

## data

Let's try to recreate this in case of a simple blog. This might not make much sense but, for the sake of the example, let's say it does 😅. The following object represents our user flow.

```js
// data.js

const views = [
  { name: 'home' },
  { name: 'about' },
  { name: 'post1' },
  { name: 'post2' },
];


export default {
  views,
  relations: [
    {
      name: 'open about page',
      from: views[0].name,
      to: views[1].name,
    },
    {
      name: 'go back home',
      from: views[1].name,
      to: views[0].name,
    },
    {
      name: 'open a post',
      from: views[0].name,
      to: views[2].name,
    },
    {
      name: 'next post',
      from: views[2].name,
      to: views[3].name,
    },
    {
      name: 'previous post',
      from: views[3].name,
      to: views[2].name,
    },
  ],
};
```

The tests will use this object to:
1. check the initial view where we're starting
2. call user actions function according to the relation name
3. check target view

## data interpreter

For drawing, I decided to use [mermaid]. Mainly because flowcharts built with it are very well structured and I needed them to be very easy to read. Also, I've been already using [mermaid] in the past so I knew what I can do with it and it matched my expectations 🙌. At this point, we need to create the **data interpreter** for our object to get a [mermaid] script from. The [mermaid] script looks like this:

```
graph TB

home--"open about page"-->about;
about--"go back home"-->home;
home--"open a post"-->post1;
post1--"next post"-->post2;
post2--"previous post"-->post1;
```

Which results with a flowchart:

{% include img/mermaid-example-1.svg %}

It might not impress you, but hopefully is to the point 😅. This is just an SVG generated with [mermaid.cli] which we could use if we decided to generate each chart manually. But we didn't. 😎 Moving on!

The data interpreter:

```js
// mermaid-interpreter.js

// flowchart declaration starts here, the `LR` stands for left-to-right direction
export default data => `graph LR 

${data.relations.map(relation =>
  `${relation.from}--"${relation.name}"-->${relation.to};`,
).join('\n')}`;
```

It takes the user-flow object and creates mermaid script joining `from` and `to` properties with named relations. We can control flowchart direction and nodes shapes as described in the [mermaid documentation](https://mermaidjs.github.io/flowchart.html), we'll use this in next steps. Now let's add some descriptions to our nodes.

```js
const views = [
  {
    name: 'home',
    description: 'Home Page',
  },
  {
    name: 'about',
    description: 'About Me',
  },
  {
    name: 'post1',
    description: 'First Post Title',
  },
  {
    name: 'post2',
    description: 'Second Post Title',
  },
];
```

The interpreter should also create the nodes declaration part of mermaid script for us to be able to see them on the flowchart:

```js
// mermaid-interpreter.js
// with separate nodes declaration and description displayed inside them

export default data => `graph LR

${data.views.map(view => `${view.name}(("${view.description}"));`).join('\n')}

${data.relations.map(relation =>
  `${relation.from}--"${relation.name}"-->${relation.to};`,
).join('\n')}`;
```

Our generated [mermaid] script:

```
graph LR

home(("Home Page"));
about(("About Me"));
post1(("First Post Title"));
post2(("Second Post Title"));

home--"open about page"-->about;
about--"go back home"-->home;
home--"open a post"-->post1;
post1--"next post"-->post2;
post2--"previous post"-->post1;
```

And the flowchart:

{% include img/mermaid-example-2.svg %}

## mermaid component

Next up, we will need something to display those charts. For this, I decided to go with [create-react-app]. It's easy to use and I needed to easily add some more functionality to the charts page afterwards. Like tabs to split them into groups or categories, or legend for more complex flows. React with its components seemed like a valid choice at this point. To start a fresh react app we need to run `npx create-react-app mermaid-react-docker`. It creates a new folder with the pre-configured react app in it. We'll need to add two packages, `prop-types` and `mermaid`. Install them with `npm i --save mermaid prop-types`. One more thing, we should delete everything from the `src` folder, we'll create our own files. 🙃 Now we're ready to write the **mermaid component** to display our flowchart.

```js
// mermaid.js

import React from 'react';
import mermaid from 'mermaid';
import PropTypes from 'prop-types';

mermaid.initialize({
  startOnLoad: false,
});

export default class Mermaid extends React.Component {
  constructor(props, context) {
    super(props, context);

    this.state = {
      chart: null,
    };
  }

  updateChart() {
    const chart = this.props.children.toString();
    const { title } = this.props;
    mermaid.render(title, chart, svg => {
      this.setState({
        chart: svg,
      });
    });
  }

  componentDidMount() {
    this.updateChart();
  }

  render() {
    const { chart } = this.state;
    return (
      <div className="mermaid" dangerouslySetInnerHTML={{ __html: chart }} />
    );
  }
}

Mermaid.propTypes = {
  children: PropTypes.node,
  title: PropTypes.node,
};
```

## react app

We need to put it all together. To render our mermaid component with diagram built from the data we just need to import everything into `index.js` of the **react app**.

```js
// index.js

import React from "react";
import ReactDOM from "react-dom";
import getChart from "./mermaid-interpreter";
import Mermaid from "./mermaid";

import data from "./data";

// ========================================

ReactDOM.render(
  <Mermaid title="mermaidjs-flowchart">{getChart(data)}</Mermaid>,
  document.getElementById("root")
);
```

At this point our project structure should look like this:

```
mermaid-react-docker
├── .gitignore
├── package.json
├── README.md
├── node_modules
├── public
│   ├── favicon.ico
│   ├── index.html
│   └── manifest.json
└── src
    ├── data.js
    ├── index.js
    ├── mermaid-interpreter.js
    └── mermaid.js
```

Let's see if everything works as expected. `npm run start` will run the react app in development mode. A flowchart exactly the same as I showed before should appear at `localhost:3000`. I hope it works for you 🤞. If not, check every file again. It should work if you followed my instructions exactly. 🤓

## docker

To be able to easily put the flowcharts app somewhere and keep it updated with the minimum effort I decided to use [docker] 🐋. If you use [dockerhub] and can integrate it with some CI, it's the best option for instant and effortless deployment. Every time I update my user-flow data I just need to push a new version of docker image to the hub and that's it. In order to build a docker image with working flowcharts app we need to create a dockerfile. It should contain a pre-built version of our app and a server (in this case it's [nginx]). Everything we need now is to create a `dockerfile` in the root of our application.

```dockerfile
FROM node:lts-alpine AS build
WORKDIR /usr/src/app
COPY . ./
RUN npm install && npm run build

FROM nginx:alpine
COPY --from=build /usr/src/app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Now, to build the image we have to run `docker build -t mermaid-react-docker .`. When it's finished building we can check if everything works as expected with `docker run -p 8080:80 mermaid-react-docker`. The flowchart should be available at `localhost:8080`.

# 🎉

[mermaid]: https://mermaidjs.github.io
[mermaid.cli]: https://github.com/mermaidjs/mermaid.cli
[create-react-app]: https://github.com/facebook/create-react-app
[docker]: https://www.docker.com
[dockerhub]: https://hub.docker.com
[nginx]: https://nginx.org