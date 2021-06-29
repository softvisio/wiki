# Markdown

## Collapsible block

```markdown
<details>
	<summary>Title</summary>

    ...content

</details>
```

## Tabs

```markdown
<!-- tabs:start -->

#### **Tab 1**

... tab 1 content

#### **Tab 2**

... tab 2 content

<!-- tabs:end -->
```

## Code blocks

To instruct `prettier` to skip fenced code block formatting place `<!-- prettier-ignore -->` just before code block:

````markdown
<!-- prettier-ignore -->
```javascript
const a = 1;
```
````

To escape ` ``` ` inside fenced code block use ` ```` ` (four or more backticks).

`````markdown
````markdown
```
...some markdown content
```
````
`````

To escape `` ` `` in inline code block use double backticks (` `` `):

```markdown
Inline code block with escaped backtick: `` ` ``
```
