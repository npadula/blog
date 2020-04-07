+++
title = "Textbook-themed webdesign"
date = "2020-03-05"
description = "Designing a minimal flat-hierarchy website from scratch with Hugo."
+++

My plan to create a personal website started out, as many others do, on [WordPress](https://en.wikipedia.org/wiki/WordPress). I was happy at first, but grew frustrated with its limitations and decided to write a website from scratch, which meant having to understand HTML and CSS at a deeper level than I had to before. I'll highlight some of my design choices and give a walkthrough of how I implemented it with the help of a fantastic website generator called [Hugo](https://gohugo.io/).

## Design goals
- **Minimal.** I want the look of the website to be clean, and focused on the content. With content I mean more than just text; figures and code, too. This emphasis on minimalism is also just to make it easier to implement programmatically and keep dependencies to a minimum. So preferably no javascript libraries or even [bootstrap](https://en.wikipedia.org/wiki/Bootstrap_(front-end_framework)). This might seem unnecessary to some, but doing this without any libraries also makes this a richer learning experience for me.
- **Responsive.** While I want the design to be primarily fine-tuned for a desktop browser, it needs to scale properly depending on the device viewport. Responsive in this context also implies compatibility between browsers and operating systems. This means [flexbox](https://en.wikipedia.org/wiki/CSS_Flexible_Box_Layout) is preferred over [grid](https://en.wikipedia.org/wiki/CSS_grid_layout).

### Textbook-theme
I was very much inspired by the layout of some science textbooks, especially Atkins' [Physical Chemistry](https://www.amazon.com/Physical-Chemistry-Julio-Atkins-Peter/dp/0198700725). I like the use of margins for both images and notes, and that some figures would extend to the margins if their widths justified it. Textbooks also strike a nice **balance between information density and readability** that I'm going for. I thought it would be a cool idea if the home page gave the impression of a table-of-contents page that you find in the beginning of textbooks, with the first "chapter" being the _about_ page of the website, and the following "chapters" linking to all posts, in order of publication date.

{{< figure
	src="atkins.png"
	link="atkins.png"
	class="aside border"
	caption="A sample page of the 8th edition of Atkins' Physical Chemistry textbook."
>}}

**Key points:**
- Pages will look like an actual piece of paper, provided the viewport is large enough. 
- One right margin, which can be used for both figures and notes.
- Home page is just an index of all posts, styled like a table-of-contents page.
- Top bar contains only one link leading to the home page.

## Using Hugo
[Hugo](https://gohugo.io/) is a [static website generator](https://en.wikipedia.org/wiki/Web_template_system#Static_site_generators) written in [Go](https://en.wikipedia.org/wiki/Go_(programming_language)). Static website generators are especially useful for creating blogs, personal pages, or documentation websites -- any [website](https://en.wikipedia.org/wiki/Static_web_page) with content that does _not_ change from visitor to visitor. It is a simple method to avoid having to write [HTML](https://en.wikipedia.org/wiki/HTML) manually. Hugo seems to be picking up steam; it has a reputation for being extremely fast, and _relatively_ easy to use. 

### Getting started
Step one is to install Hugo for whatever system we have, described [here](https://gohugo.io/getting-started/installing/). Once `hugo` is in our path, use these command line arguments:

```
hugo new site textbook
cd textbook
```

You should now have a directory called `textbook` with a few subdirectories and one file. I'll explain the significance of the ones necessary to follow this guide:

- `content`: In here you will put all your own posts in a syntax called [markdown](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet), which is mostly syntactic sugar over raw HTML.
- `layouts`: This directory contains all the HTML templates that Hugo will use to layout each post in your `content` directory.
- `static`: These files will be directly copied to your static website. Any global and unchanging files go here.
- `config.toml`: This is your configuration file where you specify site-wide variables.

{{< note title="Config file types">}}
By default, Hugo uses [TOML](https://en.wikipedia.org/wiki/TOML) for your config file, but you can also use [YAML](https://en.wikipedia.org/wiki/YAML) or even [JSON](https://en.wikipedia.org/wiki/JSON).
{{< /note >}}

#### Running Hugo
Hugo has a very useful feature where it can run as a real-time server by running the command `hugo server` from within the root of your project folder (`textbook` in our case). It will then periodically scan your files and "recompile" your website whenever it detects a change. One of Hugo's selling points is that this process is remarkably performant. All you need to do is to go to `localhost:1313` in your browser and you should see your website, provided you have already written the HTML files. 

If you run `hugo` without `server`, all files related to your website will be emitted to the directory `public`. When you're prototyping and actively editing, it makes more sense to use Hugo as a server.

{{< note title="Caching">}}
Your browser will often cache files related to your website which means that recompilation of your website will not always reflect your current changes. This is especially noticable when editing your stylesheet. You can disable this by running `hugo server --noHTTPCache`. Read more [here](https://discourse.gohugo.io/t/static-css-changes-no-updating-browser-cache-with-hugo-serve/16169).
{{< /note >}}

### General layout
We create three HTML files with the following structure:
```
└── layouts
    ├── _default
    │   ├── baseof.html
    │   └── single.html
    └── index.html
```
The `layouts/_default/` folder is important because Hugo has a [lookup order](https://gohugo.io/templates/lookup-order/) for certain files; putting it in this folder means it will be found last, implying it is the default. 

#### baseof.html
`baseof.html` plays a special role: it is used to give the _outer shell_ of each page of your website. Any code that is identical in each page should go here.

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<title>{{ if not .IsHome }}{{ .Title }} &middot; {{ end }}{{ .Site.Title }}</title>
	<meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
	<link rel="stylesheet" href="{{ relURL `/style.css` }}"/>
</head>
<body>
	<nav><a href="{{ .Site.BaseURL }}"><img src="{{ relURL `/home.svg` }}"></a></nav>
	{{ block "main" . }}{{ end }}
</body>
</html>
```

Every statement that is wrapped in between double curly braces `{{` `}}` is something that will be handled and executed by Hugo's templating library. In the `<head>` tag we have three elements:
1. The title of the browser toolbar is the post's title concatenated with the site-wide title given in your `config.toml` file. If you're at the home page the post's title is omitted, along with the concatenation character **&middot;**.
2. A `<meta>` tag which is to make sure that your phone's browser won't try to do it's own trickery.
3. The `<link>` tag references a CSS stylesheet which we have put in `static/style.css`.

And the `<body>` tag has the navigation bar which we put at the top with only one image in the middle to link to the site's home page. The next line is important: `{{ block "main" . }}{{ end }}`. It tells Hugo to look for a piece of text marked `main` in other layout files and place it here. That's what our other two HTML files do.  and `index.html` provides it for the home page.

#### single.html
`single.html` provides the `main` block for a single post page.

```html
{{ define "main" }}
<main class=single>
	<header>
		<div id=sitetitle>{{ .Site.Title }}:</div>
		<h1>{{ .Title }}</h1>
		{{ with .Page.Date }}<time>{{ .Format "2 Jan 2006" }}</time>{{ end }}
		<div id=description>{{ .Param "description" }}</div>
	</header>
	<article class=single>
		<aside id=toc>
			{{ .TableOfContents }}
		</aside>
		<span id=dropcap></span>{{ .Content }}
	</article>
</main>
{{ end }}

```

The `<header>` tag (not to be confused with `<head>`) contains the website title followed by the post's title. Then there is an optional date element, and a description or subtitle of the post. Inside `<article>` we first render the table of contents of the page, and wrap it in an `<aside>` tag so that it will move to the right margin responsively. The next `{{ .Content }}` statement calls the markdown renderer and converts your content files from markdown to raw HTML. (`<span id=dropcap></span>` is a hack that I will explain later on.)

#### index.html
`index.html` provides the layout for the home page of the website.

```html
{{ define "main" }}
<main>
	<header>
		<div id=sitetitle>{{ .Site.Title }}:</div>
		<h1>Contents</h1>
	</header>
	<article>
		<ol class=home>
		{{ range $index, $page := .Site.RegularPages }}
			<li class=home>
				<div class=number>{{ printf "%02d" $index }}</div>
				<div class=details>
					{{ with $page.Date }}<time class=home>{{ .Format "2 Jan 2006" }}</time>{{ end }}
					<a href="{{ .RelPermalink }}"><h2 class=home>{{ $page.Title }}</h2></a>
					<div class=subtitle>{{ with $page.Param "description" }}{{ . }}{{ end }}</div>
				</div>
			</li>
		{{ end }}
		</ol>
	</article>
</main>
{{ end }}
```

The `<header>` tag is fairly similar, but something more interesting happens the `<article>` tag. `range` in hugo is a command that will loop a through the elements of a given list, which, in this case, are all the pages in your `content` directory. Inside each `<li>` item some custom formatting is done so that we don't have to rely on the default `list-style`.

### Content files
In the `content` folder we have the following directory structure:

```
└── content
    ├── about.md
    └── posts
        └── example.md
```

The `about.md` page will be shown first on the home page, and then come the markdown files in `posts`. Each markdown file has a preamble called [front matter](https://gohugo.io/content-management/front-matter) which is enclosed by lines with three plug signs `+++`. (If you want to use YAML instead of TOML, you would use three minus signs `---`.) Front matter can be used to set certain variables per content file.

```toml
+++
weight = 99
date = "2019-03-11"
title = "About"
description = "Learn about this website and its author."
+++
```

Now you understand where all these variables are taken from in `single.html`. The `weight` variable determines priority of the listing in `index.html`. So by putting a high number we guarantee that the about page is shown first.

### Global settings
Then we have a few more remaining files which much be created in order to have a functioning Hugo website. We put a CSS stylesheet and the home icon in `static` folder so that it can be properly referenced in `baseof.html`. We also customize our `config.toml` and get the following directory structure:

```
├── config.toml
└── static
    ├── style.css
    └── home.svg
```

With these settings in our config file:

```toml
baseURL = "https://martigeon.github.io/textbook-hugo"
title = "My Website"
publishDir = "docs"

[permalinks]
  posts = "/:title/"

[markup.goldmark.renderer]
  unsafe = true

[markup.highlight]
  style = "pygments"
  tabWidth = 2
```

Take note of the `[permalinks]` setting. This is a little trick which makes sure that the URL goes from `example.com/posts/file-name/` to `example.com/title/` which just makes it a little bit cleaner. The `publishDir` variable is necessary to get this repository to work with [github pages](https://gohugo.io/hosting-and-deployment/hosting-on-github/#github-project-pages).

## Specific features
The exact contents of the `style.css` file is probably the most interesting, but it's too large to display here and contains a lot of boilerplate. I will go over some features which I coded into this stylesheet.

### Typography
Typography is surprisingly complicated -- both in terms of design and rendering technology. (Just look at [subpixel rendering](https://en.wikipedia.org/wiki/Subpixel_rendering), [hinting](https://en.wikipedia.org/wiki/Font_hinting), [kerning](https://en.wikipedia.org/wiki/Kerning), or [ligatures](https://en.wikipedia.org/wiki/Orthographic_ligature).) I was initially committed to using a [serif](https://en.wikipedia.org/wiki/Serif) font because this more accurately reflects the styling of a textbook, but serif fonts are also more difficult to render readably at small font sizes. [Medium.com](https://en.wikipedia.org/wiki/Medium_(website)), most notably, uses a serif font, but their content is also spaced more generously, which I wanted to avoid in the interest of a higher information density. 

{{< note title="Web typography links" >}}
- [Old but interesting summary of font-rendering](https://www.smashingmagazine.com/2012/04/a-closer-look-at-font-rendering/)
- [How to use a system font stack](https://www.wpmediamastery.com/why-and-how-to-use-a-system-font-stack/)
- [Should I use a web font?](https://medium.com/hackernoon/web-fonts-when-you-need-them-when-you-dont-a3b4b39fe0ae)
- [Web font best-practices](https://www.zachleat.com/web/comprehensive-webfonts/)
- [Notes on golden ratio, line height vs font size](https://medium.com/@zkareemz/golden-ratio-62b3b6d4282a)
- [Design Guidelines for Web Readability (2017)](http://www.atw-lab.com/public/data/webReadability_3.0.2.pdf)
{{< /note >}}

And then there was the question of whether to go for a web-font or a system font stack. This seems to be quite a contested issue among web developers. Seeing as one of my design goals was minimal dependencies, I thought it made sense to go for a fairly ubiquitous sans-serif font stack:

```css
font: normal 15px/1.4 -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
```

This also meant that I would have to use color and styling to differentiate between headings and text instead of relying on font pairings. I did this out of the assumption that it's probably more difficult to create a consistent design across platforms with _two_ font stacks.

#### Drop cap hack
By manually placing an empty inline element before the first paragraph, I got a [drop cap](https://en.wikipedia.org/wiki/Initial) effect like so:

```css
article > p:first-child:first-letter {
  color: #ce3333;
  float: left;
  font-size: 3em;
  line-height: 0.9;
  padding-right: 3px;
}
```

Without the table of contents, I could have used this selector: `article > p:first-child:first-letter`, but this way it gives me the freedom of keeping the table of contents as an optional feature for now.

### Responsive design
To make the page layout responsive I created three viewport-width _zones_ where the CSS layout was slightly different.

1. **Small:** (below 740px) No side margins and thin padding. This includes pretty much all mobile devices in portrait orientation.
2. **Medium:** (between 740px and 1060px) Thicker padding for main text, added `auto` margins, and drop-shadow paper effect.
3. **Large:** (above 1060px) Added right column for margin-notes.

Which affects the `<main>` and `<article>` tags according to the following CSS:

```css
main {
  background-color: #fff;
  padding: 50px 20px 100px;
}

@media (min-width: 740px) {
  main {
    padding: 50px 50px 100px;
    margin: 20px auto;
    max-width: 700px;
    border: solid 1px #bbb;
    box-shadow: 0 0 20px #bbb;
  }
}

@media (min-width: 1060px) {
  article.single { margin-right: 300px; }
  main.single { max-width: 1000px; }
}
```

You can see that in the largest regime the margins are only expanded for `single.html` because I thought it was unnecessary for the home page.

#### Figures
Hugo has a feature called [shortcodes](https://gohugo.io/content-management/shortcodes/) which is a method of inserting custom HTML snippets whenever markdown is not sufficient. A common one is for [figures](https://gohugo.io/content-management/shortcodes/#figure), and has a syntax like this:

```html
{{</* figure src="img.jpg" caption="Caption." class="format" */>}}
```

One of the useful attributes of the shortcode is that you can specify what classes you want your `<figure>` tag to have with the `class` parameter. This allows me to customize the formatting of images, especially in relation to the margins. 

**Classes:**
- `wide`: This will cause the figure to expand to the total width of the page, including the right margin.
- `aside`: The figure will move to the right margin by default.
- `border`: Some images with white backgrounds look better with a small border, so this allows for that.

Here is how it is implemented:

```css
figure.border img {
  border: 1px solid #bbb;
}

@media (min-width: 1060px) {
  figure.wide {
    clear: right;
    width: 900px;
  }

  aside, figure.aside {
    float: right;
    clear: right;
    margin-right: -300px;
    width: 270px;
  }
}
```

#### Margin notes
I also made a small custom shortcode in the folder `layouts/shortcodes/note.html`. The name of the HTML file corresponds to the name you will use to call it.

```html
<aside>
	{{ with (.Get "title") }}<h4>{{ . }}</h4>{{ end }}
	{{ .Inner | markdownify }}
</aside>
```

It's a little different from figure in that it has opening and closing brackets. This is because I often want to use normal, multi-line markdown syntax when creating a margin-note, so this is an easy way of doing that.

```html
{{</* note title="Optional title" */>}}
> You _can_ use **markdown** syntax here.
{{</* /note */>}}
```

Designing it like this also means that these notes will function like block elements, seeing as they are enclosed in `<aside>` tags. If you put one inside a `<p>` tag, chrome will interpret that as an error and prepend a closing `</p>` tag to the margin-note, meaning that this will always break the paragraph. I decided that I wanted these notes to act as something more substantial than a footnote, so I just use them with that in mind. My implementation also differs from the typical Tufte style in that the box is always shown, even on mobile -- it acts identical to a figure with `aside` set as its class. 

### Remarks
I think it turned out pretty well for only three layout files (+ a shortcode file) and some CSS. Now that I've done this I can't help but feel that the vast majority of static webpages are unnecessarily overengineered. Although, full disclosure, using a WordPress template would have saved me many days of not having to learn the nuances of web design. I highly recommmend anyone getting into this to really put a lot of effort in understanding how HTML elements flow and layout. A good resource is [learnlayout.com](https://learnlayout.com/). 

There's a lot of room for improvement, of course. For example, I have not put any thought into optimizing each page for search engines (SEO). Subtle things like a favicon, too. Next steps.

## Useful links
**Demo github page:** [here](https://martigeon.github.io/textbook-hugo/).

- [Best-practices for content-heavy website design](https://constructive.co/insight/designing-for-density-best-practices-for-content-heavy-website-design/)
- [normalize.css](https://github.com/necolas/normalize.css/)
- [Comparison of browser engines (CSS support)](https://en.wikipedia.org/wiki/Comparison_of_browser_engines_(CSS_support))
- http://fontfamily.io/

**Web design inspiration:**
- [gwern.net](https://www.gwern.net/Spaced-repetition)
- [gameprogrammingpatterns.com](https://gameprogrammingpatterns.com/command.html)
- [matall.in](https://matall.in/posts/vietnam/)
- [Showcase of beautiful TeX typography](https://tex.stackexchange.com/questions/1319/showcase-of-beautiful-typography-done-in-tex-friends)
- [Amazing textbook design](https://www.principiae.be/book/pdfs/TM&Th-samplepages.pdf)
- [Tufte CSS](https://edwardtufte.github.io/tufte-css/)
