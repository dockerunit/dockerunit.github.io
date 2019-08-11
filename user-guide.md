---
layout: default
title: User Guide
nav_order: 2
permalink: /user-guide/
---
<hr>
# User Guide
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

<hr>

<hr>
Here it goes the dockerunit documentation:
<hr>

Here are some examples for building:

## Tables

| Header One   | Header Two               | Header Three  |
|:-------------|:-------------------------|:--------------|
| value        | value swedish fish       | (╯°□°）╯︵┻━┻  |
| other value  | value and plenty         | ┬─┬⃰͡ (ᵔᵕᵔ͜ )    |
| value        | value with format`LOL`   | ┬─┬⃰͡ (ᵔᵕᵔ͜ )    |
| value        | value `zoute`            | ┬─┬⃰͡ (ᵔᵕᵔ͜ )    |

## Code

```java
// Java code with syntax highlighting.
@RunWith(DockerUnitRunner.class)
public class MyTestClass {

	@Test
	@Use(MyDescriptor.class, replicas=3)	
	public void myTest(ServiceContext ctx) {
	}

}
```
### Code with link

<div class="code-example" markdown="1">

[Link button](https://github.com/dockerunit/examples)

</div>

```markdown
[Link button](https://github.com/dockerunit/examples){: .btn }
```

## Lists

### Checkbox

- [ ] I am a checkbox item one
- [ ] I am a checkbox item two
- [x] goodbye, this item is done

### Definition list

<dl>
  <dt>Name</dt>
  <dd>Cervantes</dd>
  <dt>Born</dt>
  <dd>1511</dd>
  <dt>Birthplace</dt>
  <dd>Spain</dd>
  <dt>Color</dt>
  <dd>White</dd>
</dl>

### Ordered list

1. Item 1
1. Item 2
1. Item 3

### Unordered list


- Item 1
- Item 2
- Item 3

_or_

* Item 1
* Item 2
* Item 3

## Labels

Default label
{: .label }

Blue label
{: .label .label-blue }

Stable
{: .label .label-green }

New release
{: .label .label-purple }

Coming soon
{: .label .label-yellow }

Deprecated
{: .label .label-red }
