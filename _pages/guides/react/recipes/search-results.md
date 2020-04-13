---
layout: inner
title: Grove React - Customize the Search Results
lead_text: ''
permalink: /guides/react/recipes/search-results
---

# Customizing SearchResults

The default behavior of the SearchResults component is to offer a choice between a CardResult and a ListResult. In the first subsection below, you will find instructions to change the presentation of a search result in the UI. In the second subsection, there are instructions for a common use case: configuring the middle-tier to extract data from a result document (stored in MarkLogic) and pass it along as part of the search result, so that it can be displayed on the search result in the UI.

## Change the Display of Search Results

You can override the presentation of SearchResults by passing a custom resultComponent prop to `<SearchResults/>`. Your component will receive a result prop and a detailPath.

```javascript
<SearchResults resultComponent={myCustomSearchResult} />
```

Providing a `resultComponent` suppresses the out-of-the-box choice between components in the UI, which is primarily intended to showcase some possibilities. Once you provide your `resultComponent`, Search results will simply be rendered using the provided `resultComponent`.

Note that you can also provide `resultComponent` to the higher-level `<SearchView/>` and the container library's `<SearchContainer/>`, which are both parents of `<SearchResults/>` and it will be passed down.

You can use the `<CardResult/>` in this library as a starting point for your custom component, if you wish. It accepts `content` and `header` component props to make it easier to change just those parts of the card. Let's see an example of how to do this. Create your new component by copying this code into a file named for the component at `ui/src/components/CustomSearchResult.js`:

```javascript
import React from 'react';
import CardResult from './views/search/SearchResults/CardResult';
import SearchSnippet from './views/search/SearchResults/SearchSnippet';

const CustomSearchResultContent = props => {
  return (
    <div>
      <p>You got a result!</p>
      {/* Below, we are copying the card from the CardResult that shows snippets */}
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

## Adding Data from Result Documents to the Search Results

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

