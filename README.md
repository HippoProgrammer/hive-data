# hive-data
Hive data folder for Starlight and Horizon dispatches

## Contributing

Each dispatch in this repository is composed of three things: a unique ID, a template file and a metadata entry.

The unique ID is used to identify the dispatch. It should only be composed of lowercase letters, numbers and underscores. For example, the Starlight welcome dispatch's ID is `welcome_stl`.

The template file contains the content of the dispatch, that is, the BBCode that will be used to format the dispatch. However, it can also contain special formatting codes that enter headers, footers, special values, changing values, etc. - hence why it's called a *template*. 

The template file should be located in the [templates/](templates/) folder, named after the dispatch's unique ID followed by the `.tmpl` extension. For example, the Starlight welcome dispatch's template can be found at [templates/welcome_stl.tmpl](templates/welcome_stl.tmpl).

Finally, for Hive to actually publish and update a dispatch template, it needs a metadata entry in the [index.toml](index.toml) file. The metadata entry should be formatted as follows:

```
[DISPATCH ID GOES HERE]
title = "DISPATCH TITLE GOES HERE"
nation = "DISPATCH NATION GOES HERE"
category = "DISPATCH CATEGORY GOES HERE"
```

The ID should be entered between square brackets, and all other values should be between double quotes. As an example, here's the Starlight welcome dispatch's metadata entry:

```
[welcome_stl]
title = "Welcome to Starlight! (New players click here)"
nation = "starlit_constellations"
category = "Meta/Reference"
```

This tells Hive to publish a dispatch called "Welcome to Starlight! (New players click here)", with the category "Meta/Reference", on the nation "starlit_constellations" (should be entered in lowercase and with spaces replaced by underscores). The BBCode for this dispatch will be rendered from the template with ID `welcome_stl`.

For Horizon-exclusive dispatches, use the nation `sandy_wilds`. Otherwise, use `starlit_constellations`.

## Writing a template

A basic dispatch template starts from this:

```
{%- extends "layouts/horizon" -%}
{%- block body -%}

{%- endblock -%}
```

This automatically fills in the dispatch header and footer. Everything else in the dispatch should be written between the `{%- block body -%}` and `{%- endblock -%}` tags, and it will be put between the header and footer.

To select which header/footer combination to use, edit the first line. Starlight-only dispatch should use `"layouts/starlight"`, Horizon-only dispatches should use `"layouts/horizon"`, and dispatches of relevance to both regions should use `"layouts/combined"`.

This will not add a `[box]` tag, so in the likely case you want one of those you'll have to add it yourself. A common pattern for dispatches in this repository is therefore the following:

```
{%- extends "layouts/horizon" -%}
{%- block body -%}
[box]

{... Content goes here ...}

[/box]

[i]This dispatch is updated automatically.[/i]
{%- endblock -%}
```

Layouts can be viewed and edited in the [layouts/](layouts/) folder.

## Macros

Some bits of BBCode are reused often across our dispatches. Instead of rewriting them every time, they are saved as macros so that they can be invoked with one tag.

The `heading()` macro adds a heading/title element:

```
{{- heading("Hyperion Guards") -}}
```

![Heading element example](https://i.ibb.co/pvmP5SY7/Screenshot-from-2026-05-14-22-41-27.png)

The `section()` and `endsection()` macros add a section element:

```
{{- section("Why have I been mentioned?") -}}
You have been mentioned in this dispatch because you are in the World Assembly and not endorsing at least one of the following nations:
[list][*]Delegate [b][nation]Vintrel[/nation][/b]
[*]Celestial Sentinel [b][nation]Kaidan[/nation][/b][/list]
{{- endsection() -}}
```

![Section element example](https://i.ibb.co/GXDsQSk/Screenshot-from-2026-05-14-22-43-25.png)

It is worth noting that `section()` adds a `[size=120]` tag and `endsection()` just closes it (it's equivalent to `[/size]`). If you're going to be using `[size]` tags within a section, then you should first close the tag implicitly added by `section()`, and then close the tags you add yourself, which means you shouldn't add `{{- endsection() -}}` at the end.

The `divider()` macro adds a divider:

```
{{- divider() -}}
```

![Divider element example](https://i.ibb.co/XxQ0tg3Z/Screenshot-from-2026-05-14-22-47-03.png)

**To use macros, you must import them first.** You should add the import line at the beginning of the template, *before* the `extends` tag.

Example:

```
{%- from "macros/horizon" import heading -%}
{%- from "macros/general" import section, endsection, divider -%}
{%- extends "layouts/horizon" -%}
```

All macros except `heading` can be imported from "macros/general". Since the heading macro has a different version depending on the region, Horizon-only dispatches should import it from "macros/horizon", and the rest should import it from "macros/starlight".

Macros can be viewed and edited in the [macros/](macros/) folder.

## Filling in data

One of Hive's strengths is that it can easily manage changing information across many dispatches. For this reason, you should generally not hardcode values such as dispatch or image links, colors, nation names, etc.

Instead, those values are saved in files in the [parameters/](parameters/) folder. Any template can then fill them in with `{{ }}` tags.

For example, to fill in the link to our regional Discord, you use `{{ parameters.urls.discord }}`. The corresponding value can be found within the [parameters/](parameters/) folder, inside the [urls.json](parameters/urls.json) file. If at any point our discord invite link were to change, all that we would have to do is replace the value in [urls.json](parameters/urls.json), and the new invite link would automatically be filled in and updated in any template that uses `{{ parameters.urls.discord }}`.

**Here's a few examples of instances where you should use tags to fill in data:**

### Linking to existing templates

To link to the corresponding dispatch for a Hive template, use `{{ templates.ID.url }}`. For example, to link to the Starlight welcome dispatch, whose ID is `welcome_stl`: `{{ templates.welcome_stl.url }}`. This will automatically be replaced with the link to the dispatch. If the dispatch is ever recreated, any template that references it would be updated with the new URL.

### Linking to other dispatches

To link to dispatches not contained in Hive, add an entry for them in the [dispatches.json](parameters/dispatches.json) file, if they are not listed there already. Give the dispatch an ID, and then link to it using `{{ parameters.dispatches.ID }}`.

If the dispatch is ever re-created, the URL can be changed once in [dispatches.json](parameters/dispatches.json), and every dispatch that references it will be updated.

If the dispatch is ported to Hive, all you have to do is find and replace `{{ parameters.dispatches.ID }}` with `{{ templates.ID.url }}`. After that, remove the entry from [dispatches.json](parameters/dispatches.json).

### Images

If you're using images in a dispatch, do not link them directly. Add an entry for them in the [images.json](parameters/images.json) file, and then reference them with `{{ parameters.images.ID }}`. The BBCode to render the image would then be: `[img]{{ parameters.images.ID }}[/img]`.

### People / government officials

Government officials change every term. Instead of entering the current holder of a position, you should enter that position into [people.json](parameters/people.json) if it doesn't exist already, and then reference it.

For example, to fill in the current Aetherion's nation name: `{{ parameters.people.aetherion.nation }}`.

To fill in their Discord handle: `{{ parameters.people.aetherion.discord }}`.

To ping their nation and add the Discord handle next to it: `[nation]{{ parameters.people.aetherion.nation }}[/nation] ({{ parameters.people.aetherion.discord }})`.

Whenever there's a government official change, all that needs to be done is update [people.json](parameters/people.json), and any dispatch that references government officials will be updated accordingly.

### Armada ranks

Similarly to government positions, holders of Armada ranks are listed in [armada.json](parameters/armada.json).

### Colors

If you need to use a specific color for a specific thing in a dispatch, you can just enter it directly. However, for standardized colors, such as the colors of government positions, use [colors.json](parameters/colors.json) and reference them using `{{ parameters.colors.ID }}`, which will be replaced with the appropriate hex code.

Example: `[color={{ parameters.colors.flamewarden }}]Flamewarden[/color]`

### NationStates icons

If you need an icon from the [NationStates icon font](https://www.nationstates.net/page=dispatch/id=1625339), enter it wrapped in `[font=nationstates]` tags in [icons.json](parameters/icons.json) and give it a unique name. Then, you can use icons in dispatches with `{{ parameters.icons.ID }}`.

Example: `[url=https://www.nationstates.net/page=un]{{ parameters.icons.wa }} World Assembly[/url]`.

### Generated data

Some scripts can generate data that is then included in dispatches. Since this data is changed all the time, it does not live in this repository and is instead kept in a local folder and updated by the aforementioned scripts, which you can find at https://github.com/Merethin/hive-scripts.

Since the scripts and generated files require separate setup, please consult with me (Merethin) if you want to add a script and corresponding generated information. Otherwise, please do not use `{{ generated }}` tags unless they already exist and are useful for the dispatch you're trying to make. The explanation is provided here so that if you come across them, you'll know what they do.

A few examples:

Pinging new arrivals to the region in welcome dispatches - `generated.welcome.REGION`

```
[align=center][spoiler=Welcome new nations!]{% for nation in generated.welcome.starlight %}[nation]{{ nation }}[/nation]{% endfor %}[/spoiler][/align]
```

Pinging nations not endorsing regional officers - `generated.slackers.NATION`

```
{{- generated.slackers.vintrel|nation_table(8) -}}
```

*Note: the `nation_table` filter takes a list of nation names and creates a table with X columns per row (in this case 8), with `[nation][/nation]` tags for each nation listed.*