---
title: Efficiently updating RecyclerView items using payloads
description: We’ll take a look on how to use payloads together with DiffUtil to efficiently update RecyclerView items.
date: 2023-01-29 11:00:00 +0100
categories: [Android Development, Android, RecyclerView]
tags: [android, recyclerview, xml]
image: /assets/img/posts/recycler-view-payloads/payloads.gif
---

Most apps today display information to users in vertical or horizontal lists. Oftentimes, that information is dynamic and needs to be updated frequently, like the number of views, number of likes, and similar. Additionally, the list can contain images that are loaded from the network. This is why efficiently updating  `RecyclerView`  is an important aspect of having a performant app and providing a great user experience.

<hr style="width:50%; margin-left:25% !important; margin-right:25% !important; height:2px;">

> I’ve had this post written for a while now, but was debating whether to publish it or not since the topic has been covered before and Jetpack Compose is the favored UI toolkit now. However, I decided to post it in hopes someone finds this information helpful.

<hr style="width:50%; margin-left:25% !important; margin-right:25% !important; height:2px;">


## Using DiffUtil and ListAdapter

To efficiently update the  `RecyclerView`  with new data, we have to call functions  `notifyItemInserted(position: Int)`,  `notifyItemChanged(position: Int)`,  `notifyItemRemoved(position: Int),`  and similar (avoid  `notifyDataSetChanged()`  as the docs state that is inefficient and should only be used as a last resort), which notify the  `RecyclerView.Adapter`  that underlying data has changed and it  needs  to update the views to reflect the new state. Calling these functions also comes with the benefit of  `RecyclerView`  animating all of the changes.

To avoid doing all that manually, we can use  `DiffUtil`  ([official documentation](https://developer.android.com/reference/androidx/recyclerview/widget/DiffUtil)).  `DiffUtil`  is a utility class that compares old and new data, calculates the differences, and then notifies the  `RecyclerView.Adapter`  of the changes it has to make to reflect the new state. Here is an example of a  `DiffUtil.ItemCallback`  that we will use later.


<script src="https://gist.github.com/landomen/94ff483e9fff5d8912a5d0ba105c475f.js"></script>


Function  `areItemsTheSame(oldItem: Item, newItem: Item): Boolean`  checks whether two items are the same based on a unique property, like id or uuid for example. In case  the items are not the same, the adapter will know to replace the old item with the new one. When this function returns  `true`  then the function  `areContentsTheSame(oldItem: Item, newItem: Item): Boolean`  is called, where we can check whether any of the other properties of our data model has changed.

One important thing to note is that it’s recommended to calculate the  `DiffUtil`  result on the background thread as it can be quite demanding when we have a bigger dataset and it can block the main thread. We can do that either by using a Coroutine or RxJava to move the calculation to a different thread, or we can use a  `ListAdapter`  ([official documentation](https://developer.android.com/reference/androidx/recyclerview/widget/ListAdapter)).

`ListAdapter`  further simplifies our code by extending  `RecyclerView.Adapter`  and containing logic to calculate the difference on the background thread using  `AsyncListDiffer`  ([official documentation](https://developer.android.com/reference/androidx/recyclerview/widget/AsyncListDiffer)). All we have to do is pass the new data to it using the function  `submitList(list: List)`.


## Putting it all together in a sample app

We’ll take what we’ve learned so far and put it together into a simple app that displays a list of articles. Each article has a title, subtitle, a cover image loaded from a URL, and a number of comments displayed at the bottom left corner. Users can also bookmark each article by pressing the bookmark button in the top right corner of the article. In the toolbar, there are two buttons, a refresh button that updates the number of comments on articles, and a reorder button that randomly changes the order of the articles for animation demo purposes.

Below is the source code and a video of how the app looks in action.

![A sample app and reorder animations.](/assets/img/posts/recycler-view-payloads/reorder.gif)
_A sample app and reorder animations._


<script src="https://gist.github.com/landomen/11f6250a62a0cc346ea5b12f88ada50f.js"></script>


We can see that the reorder animations behave how we want them. However, let’s take a look at animations when the user bookmarks an article or refreshes the comments count.


![Updating RecyclerView items without using payloads results in a “blinking” effect.](/assets/img/posts/recycler-view-payloads/noPayloads.gif)
_Updating RecyclerView items without using payloads results in a “blinking” effect._



## Why do we get a “blinking” effect when an article is updated?

By default, when two items (in our case articles) are the same but have different contents (in our case number of comments or whether the article is bookmarked) the  `RecyclerView`  will render the new item view and then do a cross-fade between the old and new item view, which results in the »blinking« effect visible in the GIF.

This especially looks strange when darker backgrounds are involved and when the update is a result of user interaction, like bookmarking or unbookmarking an article.

## What about performance?

Besides the “blinking” animation, another issue is that we are doing a full re-bind of the item view, which is not efficient. We only want to update the number of comments or change the bookmark icon. But instead, we are re-drawing / re-rendering the whole item view again. This includes re-loading the image when that isn’t necessary.

## How can we fix this?

One solution that shows up if you are searching for “how to disable  `RecyclerView`animation” is to set  `supportsChangeAnimation = false`  on the  `RecyclerView.itemAnimator`  or simply set the  `itemAnimator = null`.

However, that would also disable all the animations when the order of the items changes or when a new item replaces an old one for example, which we don’t want. We would like to keep all the animations except for the cross-fade one when a property on an existing item changes. Additionally, this solution doesn’t address the efficiency/performance issue we mentioned.

## Payloads

We can solve both issues, animation and efficiency, by using  `payloads`. A payloadis just an object that we define, that enables us to partially update an already existing item view, instead of doing a full re-bind.

If we take a closer look at the functions in  `RecyclerView.Adapter`, there is an additional function we can override with signature  `onBindViewHolder(holder: ViewHolder, position: Int, payloads: MutableList<Any>)`. It’s an overload of the existing function  `onBindViewHolder(holder: ViewHolder, position: Int)`  function with an additional argument of  `payloads`. Taking a look at the docs for that function, we can read the following:


> If the payloads list is not empty, the ViewHolder is currently bound
> to old data and Adapter may run an **efficient partial update using
> the payload info**. If the payload is empty, Adapter must run a full
> bind. Adapter should not assume that the payload passed in notify
> methods will be received by onBindViewHolder(). For example when the
> view is not attached to the screen, the payload in notifyItemChange()
> will be simply dropped.


How can we get a change payload? If we take a closer look at functions available in `DiffUtil.ItemCallback` we’ll find that besides the existing functions `areItemsTheSame(oldItem: Item, newItem: Item): Boolean` and `areContentsTheSame(oldItem: Item, newItem: Item): Boolean` there is also another function we can override `fun getChangePayload(oldItem: Item, newItem: Item): Any?`.  
This function is called when old and new items are the same, but their contents are different. It allows us to detect which property of the item is different and to return an object based on which we will be able to partially update the item view.


<script src="https://gist.github.com/landomen/1095b845bdd30a66a22481f758aaa132.js"></script>



Here we are comparing the old and new items and when the comments count is different, we return an instance of  `ArticleChangePayload.Comments`, which is going to allow us to later know what view we have to update. We do the same for the bookmarked state, returning an  `ArticleChangePayload.Bookmark`. And in case it’s some other change, we simply call the super function, which will return  `null`, which will result in a full re-bind.

Once we have that we can now check the  `payloads`  argument in the  `onBindViewHolder`  function. This is a list because multiple updates can be merged from different threads, as mentioned in the docs:


> The payloads parameter is a merge list from  `notifyItemChanged(int, Object)`  or  `_notifyItemRangeChanged(int, int, Object)_`.


You can decide whether to handle every single list or maybe just take the last item from the list.


<script src="https://gist.github.com/landomen/f76723b85141d3319878d060fd66193c.js"></script>


In our case, we will check the last item and in case it’s of type  `ArticleChangePayload.Comments`  we will update the comments count  `TextView`  and if it’s of type  `ArticleChangePayload.Bookmark`, we will update the bookmark  `ImageButton`. It’s also important to handle the case where the  `payloads`  is empty or of an unknown type. In that case, we have to do a full re-bind by calling the original  `onBindViewHolder(holder: ViewHolder, position: Int)`  function. This is exactly what the default implementation is of the overloaded  `onBindViewHolder`function.

After adding this, we can now see that only the changed data is updated on the view and there is no blinking effect. Perfect.


![Updating RecyclerView items with payloads.](/assets/img/posts/recycler-view-payloads/payloads.gif)
_Updating RecyclerView items with payloads._


And here is the updated adapter implementation.


<script src="https://gist.github.com/landomen/79e89cc8e2a541f2191b81e0b6180189.js"></script>


## Conclusion

We took a look at how to most efficiently update the  `RecyclerView`  content using  `DiffUtil`,  `ListAdapter`, and payloads. If we have an app that shows a list of posts and we want to frequently update the number of likes or number of views, using payloads enables us to do that efficiently and only update the view that changed instead of re-inflating and re-drawing the whole item.

You can view the source code for the sample app here:  [https://github.com/landomen/recyclerview-payloads-sample](https://github.com/landomen/recyclerview-payloads-sample)