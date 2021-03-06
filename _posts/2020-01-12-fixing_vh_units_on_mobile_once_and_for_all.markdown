---
layout: post
title:      "Fixing vh units on mobile once and for all"
date:       2020-01-12 21:59:57 -0500
permalink:  fixing_vh_units_on_mobile_once_and_for_all
---


There is an annoying bug on mobile devices when trying to align elements to the bottom of the screen or when trying to get the viewport height.

Specifically there two issues at play here:

1. Any calculation that relies on the position of the bottom of the viewport - including CSS vh units, and bottom aligned items, javascript window.innerHeight or similar js methods for getting the viewport height - don't take the URL bar visibility changes into account until the scroll is finished. The exact definition of when it's finished varies by browser. In chrome it's when the page gets to a standstill in firefox it's when one leaves go of the screen (event type touchend).


2. vh units never resize based on the URL bar visibility, even after the page completely finishes scrolling. This issue started in 2011 for safari mobile on ios 6 and December 2016 for chrome mobile 56. This is easier to work around as aligning fixed/sticky content to the bottom or getting the window inner height with javascript act as ok workarounds.

An example of a workaround that works for issue one, can be found on twitter.com's mobile navigation, sliding the page up and down you'll notice that the navigation bar is pixel perfect while scrolling in one direction but when you change directions you may see a gap of one or two pixels underneath the bar.

My best guess at this point as to how twitter is getting there bottom navigation to stay in one place while the header goes in and out of view, is that they listen for touch events and calculate the current touch location vs the touchdown location. Together with prior knowledge of the navigation bar height on various devices, this data can be used to make an educated guess as to the position of the scroll bar. And used to update the bottom CSS property of the navigation element.

The navigation drawer, on the other hand, is not worth the effort as it's not that important that it be bottom aligned. Instead, a margin-bottom of 44px does the job good enough.

So that all that is a nice theory but digging through minified code can take a lot of time. What does twitter actually do?

Searching through the source code I find three references to _handleNavBarHeightChange which sounds pretty self-explanatory two of which are in blocks labeled _renderTopNav (line 34107) and _renderBottomNav (line 34135). The third one, where this is defined is at line 34233.

This intern references r.topNavNode.getBoundingClientRect() and r.bottomNavNode.getBoundingClientRect() which might be where the calculation comes from.

line 34107 (formatted by chrome dev tools):

```javascript
              l()(a()(r), "_renderTopNav", (function() {
                  var e = r.props
                    , t = e.loggedInUserId
                    , n = e.secondaryBar
                    , i = e.withTabBar
                    , o = e.withBottomLoginSignupBar
                    , a = r.state.contentMode
                    , s = !!t
                    , c = r._isTwoColumn()
                    , u = 0;
                  return r._showPrimaryBar && (u += 1),
                  (!i || r._bottomNavEnabled) && s || a !== Fe.OneColumn || o || (u += 1),
                  n && (u += 1),
                  d.createElement(d.Fragment, null, d.createElement(R.a, {
                      style: {
                          height: "calc(" + M.a.theme.componentDimensions.appBarHeight + " * " + u + ")"
                      }
                  }), d.createElement(R.a, {
                      ref: r._setTopNavRef,
                      style: [qe.topNavWrapper, r._appBarHidden && !c ? qe.hideAppBar : qe.showAppBar]
                  }, d.createElement(Le.a, {
                      isLarge: !0,
                      onLayout: r._handleNavBarHeightChange
                  }, r._showPrimaryBar ? s ? r._renderLoggedInTopNav() : r._renderLoggedOutTopNav() : null, n ? d.createElement(R.a, {
                      style: [qe.secondaryBar, c && qe.secondaryBarTwoColumn]
                  }, n) : null), r._renderPillControl()))
              }
              )),
              l()(a()(r), "_renderBottomNav", (function() {
                  var e = r.props.loggedInUserId
                    , t = r._isTwoColumn();
                  return d.createElement(Ne.a.BottomNav, null, d.createElement(Re.a, null, (function(n, o) {
                      return d.createElement(R.a, i()({
                          ref: n()
                      }, o({
                          style: [qe.bottomNavWrapper, !e && !t && qe.bottomNavContainer]
                      })), d.createElement(Le.a, {
                          isLarge: !0,
                          onLayout: r._handleNavBarHeightChange,
                          ref: r._setBottomNavRef
                      }, e ? r._renderLoggedInPrimaryNavAppTabBar() : r._renderLoggedOutBottomNav()))
                  }
                  )))
              }
              )),

```

line 34233 (formatted by chrome dev tools):

```javascript
              l()(a()(r), "_handleNavBarHeightChange", (function(e) {
                  r._reportVisibleHeight()
              }
              )),
              l()(a()(r), "_handleTransitionEnd", (function() {
                  r._reportVisibleHeight()
              }
              )),
              l()(a()(r), "_reportVisibleHeight", (function() {
                  var e = r.props.onHeightChange;
                  if (r._topNavNode && e) {
                      var t = r._topNavNode.getBoundingClientRect()
                        , n = Math.max(0, t.bottom - Math.max(0, t.top))
                        , i = r._bottomNavNode ? r._bottomNavNode.getBoundingClientRect() : {
                          top: 0,
                          bottom: 0
                      };
                      e({
                          top: n,
                          bottom: Math.max(0, i.bottom - Math.max(0, i.top))
                      })
                  }
              }
              )),
```

While logging these values to console I still only see two results depending on whether the URL bar is hidden or not. It's only called when the scroll direction changes not at every frame while the URL bar is going in and out.


Another possibility is that there is some combination of absolute, fixed, and relatively positioned elements that can show this value. Although I don't see where this would come from.

Instead, I'll settle for a good fix for issue two and leave issue one for browsers to fix.

### Fixing vh units so they can be freely used anywhere in CSS irrelevant of dom structure

1. Create a js function that sets a CSS variable to a hundredth of the current window height.

    ```javascript
    let root = document.documentElement;

    function updateRealViewportDimensions() {
      console.log(`1vh = ${window.innerHeight / 100}px`)
      root.style.setProperty('--real-vh', (window.innerHeight / 100) + "px");
    }
    ```

2. Add an event listener to any event that seems remotely related to touching scrolling or resizing the window that calls the above function.

    ```javascript
    updateRealViewportDimensions()
    const vhChangeEventTypes = [
      "scroll",
      "resize",
      "fullscreenchange",
      "fullscreenerror",
      "touchcancel",
      "touchend",
      "touchmove",
      "touchstart",
      "mozbrowserscroll",
      "mozbrowserscrollareachanged",
      "mozbrowserscrollviewchange",
      "mozbrowserresize",
      "MozScrolledAreaChanged",
      "mozbrowserresize",
      "orientationchange"
    ]
    vhChangeEventTypes.forEach(function(type) {
      window.addEventListener(type, event => updateRealViewportDimensions());
    })
    ```

3. In your CSS you can then reference the variable using the var syntax.

    ```css
    #main-navigation.--open {
      position: fixed;
      height: calc((var(--real-vh) * 100) - var(--header-height, 45px));
      width: 100vw;
      z-index: 100;
    }
    ```

All the code is available at [github.com/arye-dov-eidelman/css-real-vh](https://github.com/arye-dov-eidelman/css-real-vh)

The live demo with and without the vh fix is availble at [aryedoveidelman.com/css-real-vh](https://aryedoveidelman.com/css-real-vh/)


