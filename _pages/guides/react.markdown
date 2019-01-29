---
layout: inner
title: Grove React Developers Guide
lead_text: ''
permalink: /guides/react/
---

# Guide to Working with the Grove React UI

This is a guide for developers working with the React UI provided by Grove. It includes step-by-step guides for customizing the detail page, search results, and navbar.

## DO THIS FIRST: Read up about React

Before modifying or extending the Grove React UI, you should be familiar with React. The official React website has a [brief and well-structured introduction to React concepts](https://reactjs.org/docs/getting-started.html). Please work through it first. It will make everything else much easier to learn.

## Development Environment Setup

You may get errors in your code editor when opening React files if it does not have a JSX plugin. (You can add the "Javascript (Babel)" plugin to Sublime, for example.)

## Customization Recipes

### Customize the detail page

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

    For inspiration or a starting-point, you can [look at the implementation of the default template here](https://project.marklogic.com/repo/projects/NACW/repos/muir-core-react-components/browse/src/DetailView/DetailView.js#7-29).

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

#### Why this works

The [`<DetailView/>` component provided by grove-core-react-components](https://project.marklogic.com/repo/projects/NACW/repos/muir-core-react-components/browse/src/DetailView/DetailView.js) accepts a template prop that will override the default template when a document is successfully retrieved. It gets passed all the props that `<DetailView/>` itself received. Typically, you will write a React component to render those props as desired. It can then be passed to `<DetailView/>` like so:

```javascript
<DetailView template={myCustomDetailComponent} />
```

In out-of-the-box Grove projects based on the grove-react-template, `<DetailView/>` is the top-level 'dumb' component being wrapped by a 'smart' Redux container, `<DetailContainer/>`. [`<DetailContainer/>`](https://project.marklogic.com/repo/users/pmcelwee/repos/muir-core-react-redux-containers/browse/src/containers/DetailContainer.js) passes along the template prop to the `<DetailView/>`.

### Customizing SearchResults

The default behavior of the SearchResults component is to offer a choice between a CardResult and a ListResult. In the first subsection below, you will find instructions to change the presentation of a search result in the UI. In the second subsection, there are instructions for a common use case: configuring the middle-tier to extract data from a result document (stored in MarkLogic) and pass it along as part of the search result, so that it can be displayed on the search result in the UI.

#### Change the Display of Search Results

You can override the presentation of SearchResults by passing a custom resultComponent prop to `<SearchResults/>`. Your component will receive a result prop and a detailPath.

```javascript
<SearchResults resultComponent={myCustomSearchResult} />
```

Providing a `resultComponent` suppresses the out-of-the-box choice between components in the UI, which is primarily intended to showcase some possibilities. Once you provide your `resultComponent`, Search results will simply be rendered using the provided `resultComponent`.

Note that you can also provide `resultComponent` to the higher-level `<SearchView/>` and the grove-core-react-redux-container library's `<SearchContainer/>`, which are both parents of `<SearchResults/>` and it will be passed down.

You can use the `<CardResult/>` in this library as a starting point for your custom component, if you wish. It accepts `content` and `header` component props to make it easier to change just those parts of the card. Let's see an example of how to do this. Create your new component by copying this code into a file named for the component at `ui/src/components/CustomSearchResult.js`:

```javascript
import React from 'react';
import { CardResult, SearchView, SearchSnippet} from 'grove-core-react-components';

const CustomSearchResultContent = props => {
  return (
    <div>
      <p>You got a result!</p>
      // Below, we are copying the card from the CardResult that shows snippets
      <div className="ml-search-result-matches">
        {
          props.result.matches &&
            props.result.matches.map((match, index) => (
              <SearchSnippet match={match} key={index} />
            ))
        }
      </div>
    </div>
  )
};

const CustomSearchResult = props => {
  // 1. Suppress the header, though you could change the header instead.
  // 2. Add 'You got a result!' to each result content using the above component.
  return (
    <CardResult
      result={props.result}
      detailPath={props.detailPath}
      header={null}
      content={CustomSearchResultContent}
    />
  );
};

export default CustomSearchResult;
```

Now, you can wire in your `CustomSearchResult`. Open up `ui/src/components/Routes.js`, which includes the `<SearchContainer/>`. First, import your component:

```javascript
import CustomSearchResult from './CustomSearchResult';
```

Then, look for the `<Route/>` that renders the `<SearchContainer/>` and pass your `resultComponent` prop to it. The path, etc., may be different in your application. The key point is that you need to pass `resultComponent` to `<SearchContainer/>`.

```javascript
<Route
  exact
  path="/"
  render={() => <SearchContainer resultComponent={CustomSearchResult} />}
/>
```

#### Customizing display when no results are found

You can also override the component displayed when no results are found, by passing in a component as a `noResults` prop. This will work if you pass the `noResults` prop to the `<SearchContainer/>` as above as well, though this example shows it being passed directly to the `<SearchResults/>` component.

```javascript
const customNoResults =  () => (
  <p>Truly, there are no results.</p>
)

<SearchResults noResults={customNoResults} />
```

#### Adding Data from Result Documents to the Search Results

*Note that this section assumes you are using the out-of-the-box "all"-typed search in the Grove middle-tier, which is closely tied to the MarkLogic Search API. Grove search endpoints can be configured so that you are searching across entities at a higher level of abstraction than documents or from sources other than MarkLogic, in which case, you may need to use some other technique to return the data you require.*

The search results that are returned from the Grove middle-tier by default are very similar to the results returned by the MarkLogic Search API. If you inspect the `result` prop passed to your custom search result component, you can see its structure. You can also inspect network traffic using your browser's Javascript console to see the response to call to `/api/search`. There is a good amount of information in each search result, but it does not return the full contents of matched documents - for good reason! If it did, large documents could dramatically slow down your project.

You can extract additional information from MarkLogic's Search API by adding an `extract-document-data` search option in the saved search options referenced by `namedOptions` in your middle-tier configuration. See ["Extracting a Portion of Matching Documents"](https://docs.marklogic.com/guide/search-dev/query-options#id_37692) for details. If you don't know which namedOptions you are using:

1. **Open `middle-tier/routes/api/index.js`.** This is where you typically install and configure endpoints in your Grove middle-tier. Each endpoint created here will be nested under an `/api` prefix.

2. **Find the search route whose results you want to add the additional information to.** Out of the box, Grove includes a `/search/all` search endpoint, though it is possible to add additional typed search endpoints. Look for a section like this:

    ```javascript
    router.use(
      '/search/' + type, // this endpoint route could be different, if your team has made changes
      factory.defaultSearchRoute({ //
        // other configuration
        namedOptions: 'my-saved-search-options'
        // ...
      })
    );
    ```

3. **Change your saved search options to extract the information you need.** See MarkLogic's documentation on ["Extracting a Portion of Matching Documents"](https://docs.marklogic.com/guide/search-dev/query-options#id_37692) for details.

4. **Use the extracted information in your custom search results components.** The extracted data will be inside `props.detail.result.extracted` in your component. You should inspect that object to see how to reference it inside your custom search results component. The code will looks something like:

    ```javascript
    const myCustomHeader = props => {
      let myLabel;
      if (props.result.extracted) {
        const myLabelExtractObject = props.result.extracted.content.find(extractContent => extract.myLabelExtractKey);
        if (myLabelExtractObject) { myLabel = myLabelExtractObject.myLabelExtractKey; }
      }
      return (
        <h2>{myLabel}</h2>
      )
    }
    ```

    Though, note that you could also combine several extracts together.

### Add an image to the top Navbar

You may prefer to have an image in the top left corner of the navigation bar.  Grove's MLNavbar component was designed with this in mind and can be customized with a prop (or two if you'd like to style the image, too).

1. **Open `ui/src/components/Navbar.js`**.  A custom Navbar component that renders Grove's MLNavbar component, with links to "Search" and "Create" was provided as part of the starter Grove project.

2. **Copy the image you want to display in the Navbar to `ui/src/components`**. For this example, `Logo.png` was copied into this directory.

3. **Import the image**.  Because this project uses Webpack, you must import the image file directly into the component so that it will be included properly in the Webpack bundle.  In this example, importing a file provides a string value for the final path so that the `src` attribute of the <img/> DOM node can properly reference the file.

    ```javascript
      import myLogo from './Logo.png';
    ```

4. **Pass the image as a prop to MLNavbar**.  Add the `logo` prop, with `myLogo` as the value, to the MLNavbar child component.

    ```javascript
      <MLNavbar
        {...props}
        brandLink={brandLink}
        logo={mlLogo}
      >
    ```
    Go back to browser to inspect the result.  The default position may be suitable for your needs.  If it is not, continue to the next step.

5.  **(Optional) Style the image with an inline style**  The MLNavbar component also accepts a `logoStyle` prop.  This prop allows you to pass down an inline style object to assist with tasks such as sizing or positioning the image.  You may need to customize the styling to suit your needs and properly fit the image.  Note that a React inline style object uses camelCased properties rather than a CSS string.  The following is a simple example that restricts the size of the image and moves it slightly to align it better with the navigation options.

    ```javascript
      <MLNavbar
        {...props}
        brandLink={brandLink}
        logo={mlLogo}
        logoStyle={ {marginTop: '-5px', maxWidth: '100px', maxHeight: '45px'} }
      >
    ```

    Continue interating until you are happy with the styling.

And that's it.  You should now have your Grove application showing your custom logo.
