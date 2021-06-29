# Markdown

## Collapsible block

```md
<details>
	<summary>Title</summary>

    ...content

</details>
```

## Tabs

```md
<!-- tabs:start -->

#### **Tab 1**

... tab 1 content

#### **Tab 2**

... tab 2 content

<!-- tabs:end -->
```

## Code blocks

To instruct `prettier` to skip fenced code block formatting place `<!-- prettier-ignore -->` just before code block:

````md
<!-- prettier-ignore -->
```javascript
const a = 1;
```
````

To escape ` ``` ` inside fenced code block use ` ```` ` (four or more backticks).

`````md
````markdown
```
...some markdown content
```
````
`````

To escape `` ` `` in inline code block use double backticks (` `` `):

```md
Inline code block with escaped backtick: `` ` ``
```
