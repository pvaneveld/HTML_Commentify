## HTML_Commentify

### :link: Where

Project can be found [here](https://priceless-mirzakhani-445b23.netlify.app/)

### :mag: What?

HTML_Commentify is tool to place the class name of a HTML-tag as a comment after it is closed.
Just drag and drop HTML files on the drop zone and comments are created automagically.

### :question: Why

I often place class name comments in HTML-files to keep track of my position in the tree when working in a file.
This tool makes the process less time consuming.

### :wrench: How

The main parts of the stack are as follows:

- `Svelte`: framework of choice.
- `Svelte material UI`: [component library](https://sveltematerialui.com/) to speed up development.
- `FileReader`: Web API to handle incoming files.
- `DOMparser`: Web API to parse and work with incoming files.
