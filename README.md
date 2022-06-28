# Jira Release UI Filters and Table Sorting with Tampermonkey

[Tampermonkey](https://www.tampermonkey.net/ "Tampermonkey") is s superscript manager that allows you to write and run javascript code on specific domains/pages/paths.
It’s pretty handy to alter a site you use with your niche wants.

## Needed
[https://www.tampermonkey.net/](https://www.tampermonkey.net/)

## Installation
1. Add Tampermonkey to whichever browser you use. Here’s a video of how to install and use Tampermonkey: [https://youtu.be/8tyjJD65zws](https://youtu.be/8tyjJD65zws)
2. Copy and paste the below code into a new script.
3. Replace `your-subdomain` and `your-project-name` in the superscript config line:  
   `// @match        https://your-subdomain.atlassian.net/projects/your-project-name/versions/*`
4. Replace `your-assigne-column-name` on this line `var mainUser = 'your-assigne-column-name';` with the exact text of the label in the Assignee column.  
   ![](https://thisbailiwick.com/wordpress/wp-content/uploads/2021/10/jira-assignee-label.png)
5. Save the script and reload your Jira release page.

## Caveats
The table sorting will not work if you click between the below links. You can refresh the page to get it working.
![](https://thisbailiwick.com/wordpress/wp-content/uploads/2021/10/Screenshot-of-Firefox-10-12-21-9-14-03-AM.png)

## UserScript

```js
// ==UserScript==
// @name         Jira Release filter by name and simple column sort
// @namespace    JiraFilterByNameAndColumnSort
// @version      1.1
// @description  filter release rows by name and sort by column
// @author       Kevin Clark
// @match        https://your-subdomain.atlassian.net/projects/your-project-name/versions/*
// @require      https://cdnjs.cloudflare.com/ajax/libs/tablesort/5.2.1/tablesort.min.js
// @grant        none
// ==/UserScript==
(function () {
  'use strict';

  var mainTableCssSelector = '#ak-main-content>div:last-child>div:last-child>div>:last-child>div>table'
  var mainUser = 'your-assigne-column-name';
  var userRows = [];

  function hidePastSelected(currentName) {
    document.querySelectorAll('.name-filter.selected:not([data-name="' + currentName + '"])')?.forEach(function (link) {
      link.classList.remove('selected');
    });
  }

  function showAll() {
    hidePastSelected('All');

    userRows.forEach(function (user) {
      var wrappingElement = user.parentElement.parentElement.parentElement.parentElement.parentElement;
      wrappingElement.classList.remove('hide');
      wrappingElement.setAttribute('style', 'display: table-row');
    });
  }

  function hideOthers(currentName) {

    hidePastSelected(currentName);

    userRows.forEach(function (user) {
      var wrappingElement = user.parentElement.parentElement.parentElement.parentElement.parentElement;

      if (user.innerText !== currentName) {
        wrappingElement.classList.add('hide');
        wrappingElement.setAttribute('style', 'display: none');

      } else {
        wrappingElement.classList.remove('hide');
        wrappingElement.setAttribute('style', 'display: table-row');
      }
    });
  }

  function filterRows() {
    userRows = document.querySelectorAll(mainTableCssSelector + '>tbody>tr>td:nth-child(2)>div>div>span>a');
    var names = [];

    userRows.forEach(function (user) {
      var name = user.innerText;

      if (!names.includes(name)) {
        names.push(name);
      }

      if (user.innerText !== mainUser) {
        var wrappingElement = user.parentElement.parentElement.parentElement.parentElement.parentElement;
        wrappingElement.classList.add('hide');
        wrappingElement.setAttribute('style', 'display: none');
      }
    });
    var styles = `
        .name-filters span{
          display: inline-block;
          margin: 0 10px;
          color: #0052cc;
        }

        .name-filters:hover{
            cursor: pointer;
        }

        .name-filter.selected{
          font-weight: bold;
        }
    `;
    var linkElement = document.createElement('link');
    linkElement.setAttribute('rel', 'stylesheet');
    linkElement.setAttribute('type', 'text/css');
    linkElement.setAttribute('href', 'data:text/css;charset=UTF-8,' + encodeURIComponent(styles));

    var namesLinks = '<div class="name-filters">';
    namesLinks += '<span class="name-filter" data-name="All">All</span>';

    names.forEach(function (name) {
      namesLinks += '<span class="name-filter" data-name="' + name + '">' + name + '</span>';
    });
    namesLinks += '</div>';

    var contentWrap = document.querySelector('.css-krxk0f .ko9hdm-1');
    contentWrap.insertAdjacentHTML('beforebegin', namesLinks);
    contentWrap.insertBefore(linkElement, contentWrap.firstChild);

    var links = document.querySelectorAll('.name-filters span');

    links.forEach(function (link) {

      link.addEventListener('click', function () {
        var name = this.innerText;
        this.classList.add('selected');
        name === 'All' ? showAll() : hideOthers(name);
      });

      if (link.getAttribute('data-name') === mainUser) {
        link.classList.add('selected');
      }
    });
  }

  new Tablesort(document.querySelector(mainTableCssSelector));
  filterRows();

  const observer = new MutationObserver(function (mutations_list) {

    mutations_list.forEach(function (mutation) {

      mutation.addedNodes.forEach(function (added_node) {

        if (added_node.classList.contains('ko9hdm-0')) {
          filterRows();
        }
      });
    });
  });

  observer.observe(document.querySelector(".css-1myr7x6"), {childList: true});

})();
```


