# Adding a React App to a Wordpress Page

This is a walkthrough of our process for including a React app on almost any Wordpress page. Hosting the app elsewhere and then displaying in an iframe can be convenient, but it's also limiting in many ways. This process puts the app directly into the page's DOM, which unlocks a lot of potential that is lost when using an iframe.

We will assume the use of Vite and the Code Snippets plugin throughout this guide.

### Table of Contents

1. [Building the app](#building-the-app)
2. [FTP](#ftp)
3. [Code Snippet](#code-snippet)
4. [Page Creation](#page-creation)
5. [Tips](#tips)
   1. [Be wary of CSS](#be-wary-of-css)
   2. [Use a unique root id](#use-a-unique-root-id)
6. [Updating](#updating)
   1. [Re-build the app](#re-build-the-app)
   2. [Transfer the new build](#transfer-the-new-build)
   3. [Update the snippet](#update-the-snippet)
7. [More React Apps](#more-react-apps)
8. [Areas of Improvement](#areas-of-improvement)

## Building the app

The React app must be built in order to be hosted. "npm run build" will run the build process and create a "dist" folder in the root directory. This will contain all the static assets needed to run the app on a page.

## FTP

Open Cyberduck or another FTP client. Establish a connection to the remote server where the Wordpress installation is hosted.

Find the "ip-builds" folder in the root WordPress installation directory (the same place you find wp-content, wp-admin, etc.). Create a new folder inside that directory. Give it a suitable name (probably the name of the app). The name doesn't matter except for organization and clarity.

**Do not touch any other folders or files in the Wordpress installation directory.**

Navigate into the directory you just created. Copy the "dist" folder, created by the build process, into this directory.

Close the connection.

## Code Snippet

Navigate to the Wordpress admin page and create a new PHP Snippet with the following code:

```
<?php
function enqueue_react_app_scripts() {
	if (!is_page("<YOUR PAGE NAME>")) {
		return;
	}

    wp_enqueue_script('react-app', '/<YOUR APP'S DIRECTORY>/dist/assets/<YOUR JS FILENAME>.js', array(), '1.0', true);
    wp_enqueue_style('react-app-styles', '/<YOUR APP'S DIRECTORY>/dist/assets/<YOUR CSS FILENAME>.css', array(), '1.0', false);
}

add_action('wp_enqueue_scripts', 'enqueue_react_app_scripts');
```

Replace the page name, directory names, and filenames with those that apply to your site and app build.

This snippet will tell Wordpress to run the `enqueue_react_app_scripts` function for every page. The guard clause prevents the function from proceeding for any page other than the one you specify. If the page is correct, the function will continue to enqueue the scripts that will create and style the React app on the page.

## Page Creation

Create a new Wordpress page (or open an existing one). This page will contain the React app.

In the page editor, create a new Custom HTML block with the following code:

```
<div id="<YOUR-ROOT-ID>"></div>
```

The `id` here should match the `id` of the root div in the `index.html` of your build.

Now you can visit the front end version of the page and confirm that the React app appears and works as intended.

## Tips

#### Be wary of CSS

Since the React app exists directly on the Wordpress page, it may be affected by any CSS used by the Wordpress theme, plugins, etc. This can lead to styling conflicts, and your app may not appear the same way it does when running in isolation. Additionally, styles that you use in your app's CSS file might affect the rest of the page.

This behavior is a natural result of including a React app in a Wordpress page. Whether it's desired or not will depend on the project.

Testing the app in your actual Wordpress environment, or one that mirrors it, is essential for determining what impact this behavior will have. Consider using CSS modules to prevent your app's styles from affecting the rest of the page, and using class and ID selectors to override unwanted styles that your app receives.

If total isolation from the rest of the page is needed, you are probably in the wrong place, and an iframe-based solution will probably be needed.

#### Use a unique root id

The `id` of the root div for the React app is not set in stone. If any element on the Wordpress page happens to also have an `id` of "root", a conflict could occur due to non-unique IDs that breaks the whole app.

Using a unique, descriptive `id` makes this conflict virtually impossible. If you change it, make sure to also change the following part of `main.jsx` accordingly:

```
ReactDOM.createRoot(document.getElementById("<YOUR ID HERE>"))
```

## Updating

Currently, updating the build of the React app is a manual process. Hopefully a suitable automation will be found in the future.

#### Re-build the app

Run `npm run build` again to create a new build of your app.

#### Transfer the new build

Using FTP as described above, delete the "dist" folder created last time and replace it with the new one.

#### Update the snippet

Each new build of your app will have slightly different names for the static asset files. The PHP Snippet will need to be changed to reference the new filenames.

## More React Apps

If you want your Wordpress site to have multiple pages with React apps, you can use this general approach for each of them. For the sake of organization, you might want to modify the existing PHP Snippet instead of creating a new one for each page. For example:

```
<?php
function enqueue_react_app_scripts() {
	if (is_page("<YOUR PAGE 1 NAME>")) {
      wp_enqueue_script('react-app-1', '/<YOUR FIRST APP'S DIRECTORY>/dist/assets/<YOUR FIRST JS FILENAME>.js', array(), '1.0', true);
      wp_enqueue_style('react-app-1-styles', '/<YOUR FIRST APP'S DIRECTORY>/dist/assets/<YOUR FIRST CSS FILENAME>.css', array(), '1.0', false);
	}

	if (is_page("<YOUR PAGE 2 NAME>")) {
      wp_enqueue_script('react-app-2', '/<YOUR SECOND APP'S DIRECTORY>/dist/assets/<YOUR SECOND JS FILENAME>.js', array(), '1.0', true);
      wp_enqueue_style('react-app-2-styles', '/<YOUR SECOND APP'S DIRECTORY>/dist/assets/<YOUR SECOND CSS FILENAME>.css', array(), '1.0', false);
	}
}

add_action('wp_enqueue_scripts', 'enqueue_react_app_scripts');
```

**If you choose to use separate snippets,** you will need to use unique function names, e.g. `enqueue_react_app_1_scripts()` and `enqueue_react_app_2_scripts()`.

## Areas of improvement

This process is far from perfect. Here is a list of ways it could be improved.

- **Automation.** The manual process of deploying a new build can slow down development, especially if deployments happen frequently. Fully automating the process would solve this problem.
- **Deployment Gaps.** There is a time gap between when the new app build is uploaded and when the Snippet is updated to reference the new files. If a user visits the page during this time (very likely if the site is busy), the Snippet will be referencing files that no longer exist, causing an error screen to appear.
