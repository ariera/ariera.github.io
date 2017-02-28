---
layout: post
title: Nestable, sortable and dragable categories for a Rails project
---
In the project I'm working on we wanted to have a Category model which we wanted to be nestable. But we also liked the user to have a draggable interface to manage and rearrange the order of his categories. So we chose [awesome_nested_set](https://github.com/collectiveidea/awesome_nested_set/) for the model and [jQuery.nestedSortable](http://mjsarfatti.com/sandbox/nestedSortable/) for the UI.

It took me some time to arrange things to work properly so I wanted to share my work in case it helps anybody.

**The rest of the entry is in this gist: [https://gist.github.com/1130504](https://gist.github.com/1130504)**