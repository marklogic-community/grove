---
layout: inner
title: Grove React - Customize the Styles
lead_text: ''
permalink: /guides/react/recipes/styles
---

# Customize the Styles of your Grove React UI

## Bootstrap 3 Theming

Grove React components use [Bootstrap 3](https://getbootstrap.com/docs/3.3/) classes. You can use Bootstrap 3 themes. Out-of-the-box, the Grove React UI uses a free ["bootstrap-material-design" theme](https://cdn.rawgit.com/FezVrasta/bootstrap-material-design/gh-pages-v3/index.html). For example, you can return to the default Bootstrap theme by changing the header of `{path-to-your-grove-project}/ui/src/index.js` to comment out the "bootstrap-material-design.css" theme and uncomment the default "bootstrap-theme":

```javascript
import 'bootstrap/dist/css/bootstrap.css';
import 'bootstrap/dist/css/bootstrap-theme.css';
// import 'bootstrap-material-design/dist/css/bootstrap-material-design.css';
```

If you want to include your own bootstrap theme, import that the CSS file for that theme instead.

## Custom CSS styles

You can add global CSS styles to your Grove React UI by importing a CSS file in `{path-to-your-grove-project}/ui/src/index.js`. For example, you can put your CSS in `{path-to-your-grove-project}/ui/src/index.css` and uncomment the following line in `{path-to-your-grove-project}/ui/src/index.js`:

```
import './index.css';
```

React also allows you to style individual components that you create. [Here is a page to start discovering documentation on styling in React.](https://reactjs.org/docs/faq-styling.html) How this works is out of the scope of this guide, but anything you can do in React, you can do with Grove React.
