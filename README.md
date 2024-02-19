# logseq-logtools-custom
This is a collection of modified and extended CSS/JS originally created by [cannibalox](https://github.com/cannibalox). It contains the CSS from the [logtools](https://github.com/cannibalox/logtools) plugin as well as the forked repo's CSS and JS.

**Note: I've been using this CSS and JS and testing and fixing it, and while it's all still experimental I believe it's time to release it and let others try it out.**

## Features
* Enhanced Kanban Workflow
* Inline-gallery (stashitems)
* Parallel Blocks

### Enhanced Kanban Workflow
The Kanban implementation comes in 2 flavours: Basic and Query based.

#### Basic Kanban Board
https://github.com/elgatopanzon/logseq-logtools-custom/assets/133258481/ddd9ffa0-3f8a-4715-b50f-d608605fc68d

The basic Kanban Board workflow allows you to create 4 columns with coloured backgrounds for Todo, Doing, Done and Archive. Tasks are then created inside and manually moved around in the same way that you move Logseq blocks around. You are also in charge of properly marking items as doing/done while moving them.

This was the first implementation of Kanban I did in Logseq.

Below is the template for this board:
```
- # Kanban Board #.v-kanban #.v-kanban-w300
  template:: Kanban Board (Manual)
  template-including-parent:: true
	- #todo
	- #doing
	- #done
	- #archive
```

#### Query-based Kanban Board
https://github.com/elgatopanzon/logseq-logtools-custom/assets/133258481/e600a160-7a70-4a7d-a645-22576985cfc5

The Query-based Kanban Board works far better for project management and supports tasks on the Project level based on [this Project Management workflow](https://luhmann-logseq.notion.site/A-new-approach-to-project-management-in-Logseq-8b36dd5eb25d4b9e9882742b5ee4368e). It will pick up tasks in the page that the Kanban Board is in, as well as tasks in any other page that include the project page as a tag. This means you can control your project's Kanban Board and easily add tasks in the Daily Journal, or have cross-project tasks.

Using this board is a little different to the basic one, because you can't manually position the tasks or drag them. You have to rely on priority and a special tag `#sort0`, `#sort1` etc, to position tasks within the board. By default they are sorted by priority and then by sort tag.

Moving tasks through the workflow is done by the tasks todo/doing/done state, so all you need to do is click that to have it switch to the other boards automatically! The only exception is the Archive board, which requires the `#archive` tag and applies only to done tasks (they will move from Done to Archive).

Below is the template for this board including the queries:
```
- # Kanban Board #.v-kanban #.v-kanban-query #.v-kanban-w300
  template:: Kanban Board (Query)
  template-including-parent:: true
	- #todo
		- #+BEGIN_QUERY
{
:query [:find (pull ?b [*])
:in $ ?tag
:where
[?b :block/marker ?marker]
[(contains? #{"TODO" "NOW" "LATER" "WAITING"} ?marker)]
(page-ref ?b ?tag)
[?ref :block/name "project"]
(not [?b :block/refs ?ref])
]
:inputs [:query-page]
:result-transform (fn [result] (sort-by (fn [r] [ (get r :block/priority "Z") (get r :block/scheduled) (get r :block/content) (get r :block/deadline) ]) (map (fn [m] (assoc m :block/collapsed? true)) result)))
:breadcrumb-show? true
:table-view? false
}
#+END_QUERY
	- #doing
		- #+BEGIN_QUERY
{
:query [:find (pull ?b [*])
:in $ ?tag
:where
[?b :block/marker ?marker]
[(contains? #{"DOING"} ?marker)]
(page-ref ?b ?tag)
[?ref :block/name "project"]
(not [?b :block/refs ?ref])
]
:inputs [:query-page]
:result-transform (fn [result] (sort-by (fn [r] [ (get r :block/priority "Z") (get r :block/scheduled) (get r :block/content) (get r :block/deadline) ]) (map (fn [m] (assoc m :block/collapsed? true)) result)))
:breadcrumb-show? true
:table-view? false
}
#+END_QUERY
	- #done
		- #+BEGIN_QUERY
{
:query [:find (pull ?b [*])
:in $ ?tag
:where
[?b :block/marker ?marker]
[(contains? #{"DONE"} ?marker)]
(page-ref ?b ?tag)
[?ref :block/name "project"]
[?refarchive :block/name "archive"]
(not [?b :block/refs ?ref])
(not [?b :block/refs ?refarchive])
]
:inputs [:query-page]
:result-transform (fn [result] (sort-by (fn [r] [ (get r :block/priority "Z") (get r :block/scheduled) (get r :block/content) (get r :block/deadline) ]) (map (fn [m] (assoc m :block/collapsed? true)) result)))
:breadcrumb-show? true
:table-view? false
}
#+END_QUERY
	- #archive
		- #+BEGIN_QUERY
{
:query [:find (pull ?b [*])
:in $ ?tag
:where
[?b :block/marker ?marker]
[(contains? #{"DONE"} ?marker)]
(page-ref ?b ?tag)
[?ref :block/name "project"]
[?refarchive :block/name "archive"]
(not [?b :block/refs ?ref])
[?b :block/refs ?refarchive]
]
:inputs [:query-page]
:result-transform (fn [result] (sort-by (fn [r] [ (get r :block/priority "Z") (get r :block/scheduled) (get r :block/content) (get r :block/deadline) ]) (map (fn [m] (assoc m :block/collapsed? true)) result)))
:breadcrumb-show? true
:table-view? false
}
#+END_QUERY
```

### Inline gallery (stashitems) for enhanced Whiteboard workflow
![Screenshot_select-area_20230708174602](https://github.com/elgatopanzon/logseq-logtools-custom/assets/133258481/3c4f4b2c-614f-4a36-9510-892bc4f3f13b)


I wanted to improve my workflow with the Whiteboards feature, and in order to do that I introduced a feature I named "StashSpace" which involves using a custom tag `#stashitem` on a block and it turns that block into an inline-gallery block, and each block after it will appear in a sort of gallery-like view. 

The main idea behind this is to upload assets into Logseq and document them with tags for example `#shippingcontainer #ship #stashitem`, to create a database of tagged assets related to the tags in question.

The real power comes from the Linked References of the `#stashitem` page, which allows filtering for any number of tags. The CSS also applies to the results in the Linked References page.

![Screenshot_select-area_20230708175306](https://github.com/elgatopanzon/logseq-logtools-custom/assets/133258481/9a12e402-e341-477e-83e2-408ca92c319a)

Then, when in the Whiteboard view opening the sidebar with the `#stashitem` page becomes a filterable database of your media asset items, and dragging the blocks onto the Whiteboard places that image onto the board including the tags, and looks neat when collapsed.

![Screenshot_select-area_20230708175525](https://github.com/elgatopanzon/logseq-logtools-custom/assets/133258481/11c0472b-49d9-416c-b87a-75561dbe133a)

In the long-run this avoids copying the same asset to your Whiteboard, increases the bi-directional linking of media items when using Whiteboards, and feels more in line with the "Logseq way" of managing these kinds of items in my opinion!

### Parallel Blocks
![Screenshot_select-area_20230707112512](https://github.com/elgatopanzon/logseq-logtools-custom/assets/133258481/6de0d4b6-27b0-49f3-a0a0-6c9872a4fd02)
This is a simple way to put any blocks side by side in a column view, simply by adding the tags `#parallel-2` to the 2 blocks, or `#parallel-3` to the 3 blocks. Additionally, the tags `#parallel-small` and `#parallel-big` work to have 2 blocks side by side with one taking more space than the other, similar to having a sidebar.

Using an additional tag `#parallel-float-left` or `#parallel-float-right` allows for a block to be pinned to the left or right side. This allows content of other blocks to wrap around them rather than each block acting like a column.
