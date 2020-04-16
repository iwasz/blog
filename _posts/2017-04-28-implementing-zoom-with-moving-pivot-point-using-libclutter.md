---
ID: 480
post_title: >
  Implementing zoom with moving pivot
  point using libclutter
author: admin
post_excerpt: ""
layout: post
permalink: >
  http://www.iwasz.pl/uncategorized/implementing-zoom-with-moving-pivot-point-using-libclutter/
published: true
post_date: 2017-04-28 12:09:32
---
I'm making this post, because I struggled with this functionality a lot! I was implementing it (to the extent I was happy with) in the course of 4 or even 5 days! Ok, first goes an animated gif which shows my desired zoom behavior (check out Inkscape, or other graphics programs, they all work the same in this regard):

[caption id="" align="alignnone" width="640"]<img class="size-medium" src="http://iwasz.pl/files/zoomok.gif" width="640" height="360" /> Desired zoom functionality[/caption]

Basically the idea is that, the center of the scale transformation is where the mouse pointer is on the screen, so the object under the cursor does not move while scaling, whereas other objects around it do. My first, and most obvious implementation (which didn't work as expected) was as such:
<pre>/*
 * Point center is in screen coordinates. This is where mouse pointer is on the screen.
 * The layout is : GtkWindow has ClutterStage which contains ClutterActor scaleLayer
 * (in this function named self), which contains those blue
 * circles you see on animated gif.
 *
 * ScaleLayer is simply a huge invisible plane which contains all the objects an user
 * interacts with. User can pan and zoom it as he whishes.
 */
void ScaleLayer::zoomOut (const Point &amp;center)
{
        double scaleX, scaleY;
        // This gets our actual zoom factor.
        clutter_actor_get_scale (self, &amp;scaleX, &amp;scaleY);
        // We decrease the zoom factor since this is a zoomOut method.
        float newScale = scaleX / 1.1;

        float cx1, cy1;
        // Like I said 'center' is in screen coords, so we convert it to scaleLayer coords
        clutter_actor_transform_stage_point (self, center.x, center.y, &amp;cx1, &amp;cy1);

        float scaleLayerNW, scaleLayerNH;
        clutter_actor_get_size (self, &amp;scaleLayerNW, &amp;scaleLayerNH);
        // We set pivot_point
        clutter_actor_set_pivot_point (self, double(cx1) / scaleLayerNW, double(cy1) / scaleLayerNH);
        // And finalyy perform the scalling. Fair enough, isn't it?
        clutter_actor_set_scale (self, newScale, newScale);
}
</pre>
Here is the outcome of the above:

[caption id="" align="alignnone" width="640"]<img class="size-medium" src="http://iwasz.pl/files/zoomfail.gif" width="640" height="360" /> Zoom fail[/caption]

It is fine until you move the mouse cursor which changes the pivot point (center of the scale transformation) while scale is not == 1.0. I dunno why this happens. Apparently I do not understand affine transformations as well as I thought, or there is a bug in libclutter (I doubt it). The solution is to convert the pivot point from screen to scaleLayer coordinates before scaling (as I did), and again after scaling. The difference is in scaleLayer coordinates, so it must be converted back to screen coordinates, and the result can be used to cancel this offset you see on the second gif. Here is my current implementation:
<pre>void ScaleLayer::zoomIn (const Point &amp;center)
{
        double x, y;
        clutter_actor_get_scale (self, &amp;x, &amp;y);

        if (x &gt;= 10) {
                return;
        }

        double newScale = x * 1.1;

        if (newScale &gt;= 10) {
                newScale = 10;
        }

        scale (center, newScale);
}

/*****************************************************************************/

void ScaleLayer::zoomOut (const Point &amp;center)
{
        ClutterActor *stage = clutter_actor_get_parent (self);

        float stageW, stageH;
        clutter_actor_get_size (stage, &amp;stageW, &amp;stageH);

        float dim = std::max (stageW, stageH);
        double minScale = dim / SCALE_SURFACE_SIZE + 0.05;

        double scaleX, scaleY;
        clutter_actor_get_scale (self, &amp;scaleX, &amp;scaleY);

        if (scaleX &lt;= minScale) { return; } scale (center, scaleX / 1.1); 
} 

/*****************************************************************************/ 

void ScaleLayer::scale (Point const &amp;c, float newScale) { 
   Point center = c; float cx1, cy1; if (center == Point ()) { if (impl-&gt;lastCenter == Point ()) {
                        float stageW, stageH;
                        ClutterActor *stage = clutter_actor_get_parent (self);
                        clutter_actor_get_size (stage, &amp;stageW, &amp;stageH);
                        impl-&gt;lastCenter = Point (stageW / 2.0, stageH / 2.0);
                }

                center = impl-&gt;lastCenter;
        }
        else {
                impl-&gt;lastCenter = center;
        }

        clutter_actor_transform_stage_point (self, center.x, center.y, &amp;cx1, &amp;cy1);
        float scaleLayerNW, scaleLayerNH;
        clutter_actor_get_size (self, &amp;scaleLayerNW, &amp;scaleLayerNH);
        clutter_actor_set_pivot_point (self, double(cx1) / scaleLayerNW, double(cy1) / scaleLayerNH);
        clutter_actor_set_scale (self, newScale, newScale);

        // Idea taken from here : https://community.oracle.com/thread/1263955
        float cx2, cy2;
        clutter_actor_transform_stage_point (self, center.x, center.y, &amp;cx2, &amp;cy2);

        ClutterVertex vi1 = { 0, 0, 0 };
        ClutterVertex vo1;
        clutter_actor_apply_transform_to_point (self, &amp;vi1, &amp;vo1);

        ClutterVertex vi2 = { cx2 - cx1, cy2 - cy1, 0 };
        ClutterVertex vo2;
        clutter_actor_apply_transform_to_point (self, &amp;vi2, &amp;vo2);

        float mx = vo2.x - vo1.x;
        float my = vo2.y - vo1.y;

        clutter_actor_move_by (self, mx, my);
}
</pre>
The whole project is here : <a href="https://github.com/iwasz/data-flow-gui">https://github.com/iwasz/data-flow-gui</a>
Here's the thread which pointed me in right direction : <a href="https://community.oracle.com/thread/1263955">https://community.oracle.com/thread/1263955</a>