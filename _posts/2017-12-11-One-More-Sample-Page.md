---
title: One More Sample Page
published: false
---

Text can be **bold**, _italic_, ~~strikethrough~~ or `keyword`.

[Link to another page](another-page).

This post explores the use of Impacket’s wmiexec.py and psexec.py, detailing their functions, detection methods, and my practical experiences with them.


# [](#header-1)Understanding **Impacket**!


Impacket is a collection of Python libraries designed for network protocols like SMB, WMI, and NTLM. It is essential for conducting lateral movement, privilege escalation, and credential harvesting. In this post, I focus on two key Impacket tools: psexec.py and wmiexec.py.


## [](#header-2)What is **psexec.py**


 
psexec.py is a tool that replicates Microsoft’s PsExec functionality, allowing remote command execution over SMB. Here’s how it operates:
1.	Creates a Remote Service: Uploads a randomly named executable to the hidden ADMIN$ share on the target machine.
2.	Registers the Service: Uses RPC and the Service Control Manager to register and start the service, running with elevated privileges.
3.	Communicates via Named Pipes: After service start, it uses named pipes for data communication.
![image](https://github.com/user-attachments/assets/1daee877-76fb-4ebb-ac09-a698cae0e557)



### [](#header-3)Header 3

```js
// Javascript code with syntax highlighting.
var fun = function lang(l) {
  dateformat.i18n = require('./lang/' + l)
  return true;
}
```

```ruby
# Ruby code with syntax highlighting
GitHubPages::Dependencies.gems.each do |gem, version|
  s.add_dependency(gem, "= #{version}")
end
```

#### [](#header-4)Header 4

*   This is an unordered list following a header.
*   This is an unordered list following a header.
*   This is an unordered list following a header.

##### [](#header-5)Header 5

1.  This is an ordered list following a header.
2.  This is an ordered list following a header.
3.  This is an ordered list following a header.

###### [](#header-6)Header 6

| head1        | head two          | three |
|:-------------|:------------------|:------|
| ok           | good swedish fish | nice  |
| out of stock | good and plenty   | nice  |
| ok           | good `oreos`      | hmm   |
| ok           | good `zoute` drop | yumm  |

### There's a horizontal rule below this.

* * *

### Here is an unordered list:

*   Item foo
*   Item bar
*   Item baz
*   Item zip

### And an ordered list:

1.  Item one
1.  Item two
1.  Item three
1.  Item four

### And a nested list:

- level 1 item
  - level 2 item
  - level 2 item
    - level 3 item
    - level 3 item
- level 1 item
  - level 2 item
  - level 2 item
  - level 2 item
- level 1 item
  - level 2 item
  - level 2 item
- level 1 item

### Small image

![](https://assets-cdn.github.com/images/icons/emoji/octocat.png)

### Large image

![](https://guides.github.com/activities/hello-world/branching.png)


### Definition lists can be used with HTML syntax.

<dl>
<dt>Name</dt>
<dd>Godzilla</dd>
<dt>Born</dt>
<dd>1952</dd>
<dt>Birthplace</dt>
<dd>Japan</dd>
<dt>Color</dt>
<dd>Green</dd>
</dl>

```
Long, single-line code blocks should not wrap. They should horizontally scroll if they are too long. This line should be long enough to demonstrate this.
```

```
The final element.
```
