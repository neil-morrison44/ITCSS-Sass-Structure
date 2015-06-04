# STV Sass
Based on Harry Roberts' concept ITCSS. For a complete overview, watch [this talk](https://www.youtube.com/watch?v=1OKZOV-iLj4) and [read these slides](https://speakerdeck.com/dafed/managing-css-projects-with-itcss). The files in this git serve as a working example, however they aren't perfect and haven't been adapted to work with the current STV codebase. 

Along with a basic introduction to ITCSS, this readme will also contain a few general best practices/methods for working with Sass.

*A lot of things discussed here will be common sense to some, but it's important to write down just to make sure we're all on the same page. There are also some opinionated points which may need to be discussed and ironed out by the team before the system is considered.*

## Contents
1. [Overview](#overview)
1. [1. Settings](#1-settings)
  - Variable names
  - Variable units
1. [2. Tools](#2-tools)
1. [3. Generic](#3-generic)
1. [4. Base](#4-base)
1. [5. Objects](#5-objects)
1. [6. Components](#6-components)
1. [7. Trumps](#7-trumps)
1. [Example](#example)
  - Avoiding @extend
  - Use of classes
1. [Using widgets](#using-widgets)
1. [Going further](#going-further)

## Overview
ITCSS is a certain way of structuring Sass files to minimise rewriting/undoing code, and to maximise scalability. IT stands for Inverted Triangle, which is the basis of the code structure. In our master Sass file, rules which broadly affect elements on the page are imported at the top. As specificity grows, the further down the file the rules will be imported. 

![Specificity triangle](http://i.imgur.com/okdOFdK.png)

At the top, far-reaching rules (unclassed selectors) e.g.
``` sass
body { font-family: Helvetica, Arial, sans-serif }
```

Further down the list, less specific rules will come into place (classed selectors) e.g.
``` sass
.component { background-color: red; padding: 10px; }
```

And at the bottom of the list, very specific rules are placed (classed selectors, !important can be used) e.g.
``` sass
.bg--alpha { background-color: green }
```

By keeping this top-down structure, rewriting and undoing css will be minimised as the order which the rules are placed will dictate specificity. Use of !important should be unnecessary, other than perhaps in helper files at the _very bottom_ of the list.

The full Sass structure looks like the following:

1. **Settings**   	- Variables, config.
1. **Tools** 		    - Mixins, functions
1. **Generic** 	  	- Normalize, reset, * {}
1. **Base** 		    - Unclassed HTML rules
1. **Objects** 	  	- Cosemetic-free design patterns
1. **Components** 	- Chunks of UI
1. **Trumps**   		- Helpers and overrides

## 1. Settings
Site-wide variables such as margin sizes, color schemes, font families/sizes etc. should go in here. There should be minimal 'Magic Numbers' throughout the code. **Note:** These settings are for site-wide variables. Component specific settings should go at the top of the component's file.

Example Structure:
``` 
1-settings
  _breakpoints.scss
  _fonts.scss
  _palette.scss
```

#### Variable names
Keep variable names ambiguous to prevent refactoring code in the future.

##### Bad
``` sass
// 1-settings/_pallete.scss
$red: #ff2134; // Site's primary theme color
// ...

// 6-components/_puff.scss
.puff{
  font-color: $red;
}
```
If we want to change the site's primary theme colour to green, we'll need to change it in the settings file, as well as the components file (and all other files it's used in.)

##### Good
``` sass
// 1-settings/_pallete.scss
$red: #ff2134; // Use concrete names to describe variables that won't change
$palette--primary: $red; // Use ambiguous names for variables that may change, and that are used throughout the codebase
// ...

// 6-components/_puff.scss
$puff-font-color: $palette--primary !default;
.puff{
  font-color: $puff-font-color;
}
```
This way may seem more complex than the first, but it means if we want to change the site's primary theme colour to green we'll only need to change it once within the settings file. This gives us a lot more flexibility.

**A final naming convention/structure is still to be decided**

#### Variable units
All absolute sizes should be stated in px, but later converted to em/rem with a Sass function. e.g.
``` sass
// 1-settings/_content-structure.scss
$base-spacing-unit: 20px;
// ...

// 6-components/_puff.scss
$puff-margin-bottom: $base-spacing-unit !default;

.puff{
  margin-bottom: rem($puff-margin-bottom);
}
.puff--small{
  margin-bottom: rem($puff-margin-bottom / 2);
}
```
Using pixels allows us to match designs with greater detail and will keep math as simple as possible. Converting to em/rem when it's finally needed will keep the sites responsive and accessible.

## 2. Tools
Self explanatory; keep any mixins/functions in here. High up in the list as these tools will be used throughout the rest of the Sass codebase.

Example Structure:
``` 
2-tools
  _media-queries.scss
  _units.scss
```

## 3. Generic
Generally 'set and forget' type rules will go in here, e.g. normalize, clearfix, *{}

Example Structure:
``` 
3-generic
  _clearfix.scss
  _generic.scss
  _normalize.scss
```

## 4. Base
Site-wide rules should be set here. A general rule here is there should only be tag selectors, no class selectors (although exceptions can be made.)

Example Structure:
``` 
4-base
  _fonts.scss
  _global.scss
  _headings.scss
```

Example rules: 
``` sass
// 4-base/_global.scss
html{
	font-family: $base-font-family;
	font-size: $base-font-size;
	line-height: $base-line-height;

	min-height: 100%;
}

body{
    background-color: $palette--body-bg;
    -webkit-font-smoothing: antialiased;
}

// 4-base/_headings.scss
h1{
    font-size: $h1-size;
}
h2{
    font-size: $h2-size;
}
h3{
    font-size: $h3-size;
}

h1, h2, h3, h4, h5, h6 {
	line-height: $base-line-spacing--header;
}
```

## 5. Objects
Reusable objects should be placed here, generally layout structures. **No cosmetic styling** should go in here.

Example Structure:
``` 
5-objects
  _grids.scss
  _media.scss
  _slider.scss
```

Example rules: 
``` sass
// 5-objects/_media.scss
.media{
    @extend .cf;
    display:block;
}
    .media__img{
        float:left;
        margin-right:rem($base-spacing-unit);

        &.is-half-spaced{
            margin-right:rem($base-spacing-unit / 2);
        }
    }
    
    .media__body{
        overflow:hidden;
    }
```
Since we will be reusing certain components/widgets across many sites with different themes, declaring any cosmetic or unnecessary styles here will inevitably be overwritten, causing waste. To avoid this, use this section to make minimal structures that generally won't change and can be reused across multiple widgets/components/sections. These can be further styled to suit the specific component later in a new file in the components section. (see [my example](#example) below.) 

In theory, once an object is made here, it should rarely need edited again.


## 6. Components
The majority of the site's styling will go in here. This section may grow quite large depending on the complexity of the site so further organisation into deeper folders may be needed. Caution should be exercised when doing this, however, as we should aim to make components modular and reusable when possible.

Example Structure:
``` 
6-components
  _breadcrumbs.scss
  _banner.scss
  _header.scss
  _puff.scss
  templates
    _contact-page.scss
```

**Note:** Components may depend on objects to complete the style, but two components should *rarely* rely on each other as they should be kept as completely seperate entities. If you find that you can extend one component to create a new but similar one, consider refactoring the component into an object which can be reused and later embellished in the component. Alternatively, try combining the components into a single one and using modifier flags ('.component**--alt-style**') to differentiate.

##### Bad
``` sass
// 6-components/_header-navbar.scss
.header-navbar {
  height: 60px;
  padding: 10px;
  background-color: red;
  // ...
}

// 6-components/_footer-navbar.scss
.footer-navbar {
  @extend .header-navbar;
  background-color: green;
}
```

##### Good #1
``` sass
// 6-components/_navbar.scss
.navbar {
  height: 60px;
  padding: 10px;
  // ...
}

.navbar--header {
  background-color: red
}

.navbar--footer {
  background-color: green;
}
```

##### Good #2
``` sass
// 5-objects/_navbar.scss
.navbar {
  height: 60px;
  padding: 10px;
  // ...
}

// 6-components/_header-nav.scss
.header-nav {
  background-color: red
}

// 6-components/_footer-nav.scss
.footer-nav {
  background-color: green;
}

// index.html
<div class="header-nav navbar">
  <!-- ... -->
</div>

<div class="footer-nav navbar">
  <!-- ... -->
</div>
```

One thing to note about the 'Good #1' example is that adding modifiers can quickly build up and make a single component overly complex. If this looks like it could happen, it may be best splitting the component up into multiple components like in the 'Good #2' example.

Both of the above examples work well because no component depends on another. This is not to say components can't be nested, but a single element shouldn't have two different components classes assigned to it. 

## 7. Trumps
Rules that are added at the very end, generally used for helper classes.

Example Structure:
``` 
// 7-trumps
  _helper.scss
  _palette.scss
```

Example rules: 
``` sass
// 7-trumps/_palette.scss
.bg--alpha {
  background-color: $palette--primary;
}

.txt--beta {
  color: $palette--secondary;
}
```
Since this file is at the very bottom of the list, adding the '.bg--alpha' class to an element will 'trump' any previously set background-color. While we should strive to keep specificity as low as possible, i.e. single class selectors via BEM, we will always have cases where nested classes are needed. To allow trumps to work on these, we need to add !important to the trump rules. This should be safe to do as there isn't much scope for problems this late on in the file.

## Example

``` sass
// index.html
<ul class="hor-list breadcrumbs">
  <li class="hor-list__item breadcrumbs__item">Link 1</li>
  <li class="hor-list__item breadcrumbs__item">Link 2</li>
  <li class="hor-list__item breadcrumbs__item">Link 3</li>
</ul>

// 5-objects/_lists.scss
.list-hor{
    padding: 0;
    margin: 0;
    list-style: none;
}
    .list-hor__item{
        display: inline-block;
    }
    
// 6-components/_breadcumbs.scss
.breadcrumb{
	background-color: #000;
	width: 100%;
	padding: rem($base-spacing-unit / 2) rem($base-spacing-unit);
}
	.breadcrumb__item{
		font-size: rem(11px);
		text-transform: uppercase;
		color: #fff;
		margin-right: rem($base-spacing-unit / 2);

		&:after{
			position: absolute;
			content: "|";
			color: $palette--primary;
			margin-left: rem($base-spacing-unit / 2);
		}
	}
```

Note that '.list-hor' has no cosmetic styling (color, font-size etc), while '.breadcrumb' does. This allows us to reuse '.list-hor' without constantly having to *undo* it's rules. Also note that the padding on '.list-hor' is being overwritten by the '.breadcrumb' class. Since our component comes after the object in the master Sass file, the '.breadcrumb' rules will overwrite the '.list-hor' rules without any hassle with specificity (i.e. no need for !important).

#### Avoiding @extend
[This article by Oliver Jash](http://oliverjash.me/2012/09/07/methods-for-modifying-objects-in-oocss.html) explains clearly the cons of using @extend instead of defering this functionality to the HTML. I'll highlight a few reasons not to use @extend.

My example above could have been achieved by entering only the 'breadcrumb' class in the markup and then extending the 'list-hor' class within Sass. e.g.

``` sass
.breadcrumb {
  @extend .list-hor;
  background-color: #000;
  //  ...
}
```

There are multiple reasons to avoid this:

1. By extending .list-hor, the .breadcrumb class is being hoisted up next to .list-hor...
  ``` sass
  // compiled css
  .list-hor, .breadcrumbs {/*...*/}
  ```
  We now have unrelated rules scattered across the compiled css file which has broken the IT structure, as we now have a component class mixed in with an object class. While this won't be an issue 95% of the time, a complex piece of code may cause unexpected issues elsewhere.
  
2. Extending a class with nested rules can create lots of unnecessary code. ![poorly compiled css](https://pbs.twimg.com/media/B8mlqv_CUAAi7Qg.png:large) Nesting in general should be avoided as much as possible, but if it is necessary, **never** extend it to another class. 

##### Use mixins instead of @extend
The thought of this scared me at first because it gives the illusion of waste, but it makes a lot of sense when explained. [Harry talks in depth about this here](http://csswizardry.com/2014/11/when-to-use-extend-when-to-use-a-mixin/#when-to-use-a-mixin) so I won't go into much detail. The take-home points are:
- Less specificity issues like I mentioned above.
- Gzip catches and compresses the 'wasted' code.
- While the compiled code isn't DRY, the source code is, and that's what matters.

#### Use of classes
Classes should be used to style every element where possible. Sass makes it very easy to nest rules within each other, causing unnecessary specificity. Avoid this unless you have no control over the markup.

##### Bad
``` sass
// 5-objects/_blocklist.scss
.block-list{
    padding: 0;
    margin: 0;
    list-style: none;
    
    li{
      display: inline-block;
      padding: 10px;
    }
}
```
The above code gives the 'li' element a high specificity. This means when I come to reuse that code for a component, I'll have to create more high spcificity rules, or use !important.

``` sass
// index.html
<ul class="nav block-list">
  <li class="nav__item">Item</li>
</ul>

// 6-components/_nav.scss
.nav {
  
}
  .nav__item {
    padding: 20px; // won't work
    padding: 20px !important; // will work but causes even higher specificity
  }
```
Trying to overwrite the padding won't work because '.blocklist li' has a higher specificity than '.nav__item', therefore !important is needed.

##### Good
``` sass
// index.html
<ul class="nav block-list">
  <li class="nav__item block-list__item">Item</li>
</ul>

// 5-objects/_blocklist.scss
.block-list {
  padding: 0;
  margin: 0;
  list-style: none;
}
  .block-list__item {
    display: inline-block;
    padding: 10px;
  }
    
// 6-components/_nav.scss
.nav {
  
}
  .nav__item{
    padding: 20px; // Works!
  }
```
The padding can now be changed because '.block-list__item' and '.nav__item' have the same specificity (i.e. one class). '.nav__item' will overwrite because it's a component and therefore exists later in the compiled css.

If you don't have access to the markup and therefore must nest selectors to target elements, try not to be too broad with your selectors, like the following:
``` sass
.header{
  a {
    color: red;
  }
}
```
The rule above will affect *all links in the header*. Can you guarantee they should all be red? If not, you will now have to write more code to change the other links back possibly causing unnecessary waste and even more high specificity rules.

## Using widgets
**This section is merely an idea, and may be unncessary if the team decides so. However, it should be considered as it will keep the codebase DRY and will minimlise the constant rewriting of a commonly used widgets.**

At STV we make good use of reusable 'widgets' which allow us to drop in some functionality with minimal effort. To extend this drop-in functionality, we can use certain methods to make use of a 'standard style', which can be quickly updated and extended in our individual site's Sass codebase.

To give an example, take the following mocked up 'article-list' widget. 

![Default article-list style](http://i.imgur.com/iP9cJcL.png)

The Sass code for this looks like the following (and it lives elsewhere, e.g. 'core.stv.tv/public/assets/source'):

``` sass
// $path-to-external-sass: 'core.stv.tv/public/assets/source/'
// #{$path-to-external-sass}/widgets/article-list.scss

// naming convention to be determined
$article-list__border-color: 	#ccc !default;
$article-list__spacing: 		if(variable-exists(base-spacing-unit), $base-spacing-unit / 2, 10px) !default;
$article-list__bg-color: 		#fff !default;
$article-list__font-color: 		if(variable-exists(palette--primary), nth($palette--primary, 2), red) !default;
$article-list__img-width: 		50px !default;

.article-list {
	border: solid 1px $article-list__border-color;
}
	.article-list__item {
		padding: rem($article-list__spacing);
		border-bottom: solid 1px $article-list__border-color;
		background-color: $article-list__bg-color;

		&:last-child {
			border-bottom: none;
		}
	}

	.article-list__img {
		width: rem($article-list__img-width);
		height: auto;
		margin-right: rem($article-list__spacing);
		float: left;
	}

	.article-list__body {
		color: $article-list__font-color;
	}

```
Take note of the variables at the top. They are all trailed with !default, which means we can override them. Also notice the if statements in some of the variables, acting like a ternary statement. This allows us to look for a standard variable name and use it if it exists in our site's Sass codebase, otherwise a default value is used.

Now, we need to use this widget on our new site and match it to it's theme. Our first instinct would be to copy/paste a previous style and hack away, or even just rewrite the style from scratch. But with this method, all we need to do is import the standard widget style and change some variables. We can also add some overwritten/additional classes to the end if the standard file can't fully adapt. So the Sass file in our site's codebase looks like:

``` sass
// 6-components/_article-list.scss

// configure widget with our site specific settings
$article-list__border-color: 	#555;
$article-list__spacing: 		20px;
$article-list__bg-color: 		#222;
$article-list__font-color: 		#fff;
$article-list__img-width: 		100px;

// import the 'standard style'
@import '#{$path-to-external-sass}/article-list';

// make necessary amends.
.article-list__img {
    border-radius: 50%;
}

```
This additional code can quickly transform the standard style into something completely new, with minimal additional code, keeping our global codebase DRY and modular. 

![Themed article-list style](http://i.imgur.com/ybXmCiD.png)

To clarify, our local site's widget scss file would take on the following structure:
- Settings relevant to the site's theme
- Import the standard widget style, which contains !default variables
- If necessary, list additional or overwritten rules for the widget

If the widget styling is entirely different to the standard style, there's obviously no need to import the original file. It may be a good idea to implement multiple standard 'views' for the widget, for example 'article-list--vert.scss' and 'article-list--hor.scss', then import the most relevant standard file.

**Note:** Using global/widget files would mean creating a standard file and almost never editing it again, as it will have a knock-on effect on multiple sites. If we need to make the same change across multiple sites then this structure will make that easy. If a new site project is the first to use a certain widget, create it in the widgets folder so it can be reused, and link to it in the site's file like above.

#### Using other global Sass code
The method above works well for Widgets, but it can also be used for other parts of the Sass code. Take settings files for example, we can have global settings for things like STV's colour scheme, and standard fonts, then include them using the method above and overwriting any differences.

``` sass
// core.stv.tv/public/assets/source/global/styles/settings/_fonts.scss

$base-font-size: 		16px !default;
$base-line-height: 		1.6 !default;

$mega-size: 			55px !default;
$kilo-size:      	   	28px !default;

$h1-size:        	   	50px !default; 
$h2-size:        	   	25px !default; 
$h3-size:        	   	20px !default; 
$h4-size:        	   	17px !default; 
$h5-size:        	   	15px !default; 
$h6-size:      	  	   	13px !default; 

$milli-size:   		   	12px !default;
$micro-size:   		   	10px !default;

$base-font-family: 				"FS Me Web Bold", stv-ssp, Helvetica, Arial, sans-serif !default;
$base-font-family--icons:		'icomoon' !default;

```

``` sass
// 1-settings/_fonts.scss

$base-line-height: 		1.4;
$base-font-family: 		"Open Sans", sans-serif;

@import '#{$path-to-external-sass}/settings/fonts';

```
Again, this just allows us to keep common styles between sites consistent, modular and DRY.

## Going further
ITCSS is merely a way of doing things. It's not a framework, syntax or technology that we need to commit to (with the exception of Sass but even that isn't necessary for ITCSS). It simply gives us a platform on which to easily grow STV's style codebase with minimal headaches, as we all know how incredibly messy and complex CSS can/will end up.

This means we can lay other specific methodologies on top, as we see fit. Some things to look into:

- [Sassline typography](https://sassline.com/)
- [Grids](https://github.com/csswizardry/csswizardry-grids)
