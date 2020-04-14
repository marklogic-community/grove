---
layout: inner
title: Grove React -  Customize the Detail Page
lead_text: ''
permalink: /guides/react/recipes/detail
---

# Customize the detail page

This recipe starts with the basic steps to get it done, for the busy. Then, there is a section explaining why it works, which could be worth knowing.

1. **Create a React Component that renders the view you wish to see.**

    This Component will receive the following props: `detail`, which contains the document or entity that is being viewed, in json or xml; `contentType`, for the contentType of that document or entity (for example, 'application/json' or 'application/xml; `label`, which is human-readable; and `id`, which is not.

    To match the common organization of Grove React projects, we recommend creating this component in a file named for your component in `ui/src/components/MyDetailTemplate.js`. Of course, you don't actually have to call your template MyDetailTemplate if you don't want. Standard practice within the React community is to assign your component to a variable and then export it at the bottom of the file:

    ```javascript
    import React from 'react';

    const MyDetailTemplate = (props) => {
      return (
        <div>
          detail: {props.detail}<br/><br/>
          contentType: {props.contentType}<br/><br/>
          label: {props.label}<br/><br/>
          id: {props.id}<br/><br/>
        </div>
      );
    };

    export default MyDetailTemplate;
    ```

    You will almost certainly want to build this template component up iteratively, so once you have a placeholder, jump to step 2 to install it.

    For inspiration or a starting-point, you can [look at the implementation of the default template here](https://github.com/marklogic-community/grove-react-ui/blob/2c33e87d9c72882bfd5bd1801ff95dfb127fb8bc/src/components/views/detail/DetailView.js#L7).
    
    When creating your component, you have the full power of the React framework and ecosystem. [Please learn about React, if you haven't already.](https://reactjs.org/docs/hello-world.html)

2. **Pass your template to the DetailContainer.**

    Open up `ui/src/components/Routes.js`. This includes a `<DetailContainer/>`. You can attach the one you created by, first, importing it:

    ```javascript
    import MyDetailTemplate from './MyDetailTemplate';
    ```

    Secondly, pass your component to `<DetailContainer/>`, keeping other props in place (and NOTE that you can pass any props you wish in here, and they will be available in your custom MyDetailTemplate component):

    ```javascript
    <DetailContainer
      template={MyDetailTemplate}
      id={id}
    />
    ```

## Why this works

The [`<DetailView/>` component](https://github.com/marklogic-community/grove-react-ui/blob/2c33e87d9c72882bfd5bd1801ff95dfb127fb8bc/src/components/views/detail/DetailView.js) accepts a template prop that will override the default template when a document is successfully retrieved. It gets passed all the props that `<DetailView/>` itself received. Typically, you will write a React component to render those props as desired. It can then be passed to `<DetailView/>` like so:

```javascript
<DetailView template={myCustomDetailComponent} />
```

In out-of-the-box Grove projects based on the grove-react-template, `<DetailView/>` is the top-level 'dumb' component being wrapped by a 'smart' Redux container, `<DetailContainer/>`. [`<DetailContainer/>`](https://github.com/marklogic-community/grove-react-ui/blob/2c33e87d9c72882bfd5bd1801ff95dfb127fb8bc/src/containers/DetailContainer.js) passes along the template prop to the `<DetailView/>`.
