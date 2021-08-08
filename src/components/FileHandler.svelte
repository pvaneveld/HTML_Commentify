<script lang="ts">
  import Textfield from "@smui/textfield";
  import Button from "@smui/button";
  import Tooltip, { Wrapper } from "@smui/tooltip";
  import Snackbar, { Label } from "@smui/snackbar";

  const fileReader = new FileReader();
  const domParser = new DOMParser();

  let dragOver = false;
  let downloadDone = false;
  let fileArray: { file: File; name: string }[];
  let fileIndex = 0;
  let commentCount = 0;
  let fileName: string;
  let commentThreshold = 1;
  let snackbarText: string;
  let snackbar;

  /**
   * @description listen to file changes and send them to the reader
   */
  const handleFileChange = async (e: DragEvent) => {
    commentCount = 0;
    dragOver = false;
    fileArray = getFileArray(e);

    if (fileArray.length) {
      fileReader.readAsText(fileArray[0].file);
    }
  };

  // set the current file name
  fileReader.onloadstart = () => (fileName = fileArray[fileIndex].name);

  // commentify file and download when done
  fileReader.onload = () => commentify(fileReader.result as string);

  fileReader.onloadend = () => {
    // reset when all files are parsed
    if (fileIndex + 1 === fileArray.length) {
      reset();
    } else {
      // up the index and go to the next file
      fileIndex++;
      fileReader.readAsText(fileArray[fileIndex].file);
    }
  };

  fileReader.onerror = () => {
    snackbarText = `Oops, something went wrong parsing the file ${fileName}`;
    snackbar.open();
    reset();
  };

  const reset = () => {
    fileIndex = 0;
    fileArray = null;
    downloadDone = true;
  };

  /**
   * @description get html files and file names from the drag event
   */
  const getFileArray = (e: DragEvent) => {
    try {
      return Array.from(e?.dataTransfer?.items ?? [])
        .filter(({ kind, type }) => {
          if (kind === "file" && type === "text/html") {
            return true;
          }
          throw "Oops, only html files are allowed.";
        })
        .map((file) => file.getAsFile())
        .map((file) => ({
          file,
          name: file.name,
        }));
    } catch (error) {
      snackbarText = typeof error === "string" ? error : error.message;
      snackbar.open();
    }
  };

  /**
   * @description place comment tags in html file
   */
  const commentify = (htmlString: string) => {
    const document = domParser.parseFromString(htmlString, "text/html");
    removeComments(document);
    document.querySelectorAll("*").forEach(insertComments);

    download(fileName, document.documentElement.innerHTML);
  };

  /**
   * @description clean up the current comments
   */
  const removeComments = (document: Document) => {
    const nodeIterator = document.createNodeIterator(
      document.body,
      NodeFilter.SHOW_COMMENT
    );

    // Remove all coments
    while (nodeIterator.nextNode()) {
      const commentNode = nodeIterator.referenceNode;
      commentNode.parentNode.removeChild(nodeIterator.referenceNode);
    }
  };

  /**
   * @description insert comment with class names after element
   */
  const insertComments = (el: Element) => {
    const className = el.className;

    if (
      className &&
      el.children.length >=
        (isNaN(Number(commentThreshold)) ? 1 : Number(commentThreshold))
    ) {
      commentCount++;
      el.parentNode.insertBefore(getComment(el), el.nextSibling);
    }
  };

  /**
   * @description create comment element containing al the classnames
   */
  const getComment = (el: Element) => {
    return document.createComment(
      `${Array.from(el.classList)
        .map((className) => `.${className} `)
        .join("")}`
    );
  };

  /**
   * @description download the edited file
   */
  const download = (filename: string, text: string) => {
    var element = document.createElement("a");
    element.setAttribute(
      "href",
      "data:text/html;charset=utf-8," + encodeURIComponent(text)
    );
    element.setAttribute("download", filename);

    element.style.display = "none";
    document.body.appendChild(element);

    element.click();

    document.body.removeChild(element);
  };
</script>

<Snackbar leading bind:this={snackbar}>
  <Label>{snackbarText}</Label>
</Snackbar>
<Wrapper>
  <div class="dropZone__childNodes">
    <Textfield
      bind:value={commentThreshold}
      style="width: 13em"
      input$min="0"
      input$max="10"
      label="Minimum number of child nodes"
      type="number"
    />
  </div>
  <Tooltip
    >The minimum amount of child nodes required for a comment to be created.</Tooltip
  >
</Wrapper>
<div
  class:dragOver
  class="dropZone"
  on:drop|preventDefault={handleFileChange}
  on:dragover|preventDefault={() => (dragOver = true)}
  on:dragleave|preventDefault={() => (dragOver = false)}
  on:dragend|preventDefault={() => (dragOver = false)}
>
  {#if !downloadDone}
    <p class="dropZone__introText">HTML-files inbound &#129666;</p>
    <img src="/assets/files-empty.svg" class="dropZone__image" alt="" />
  {:else}
    <div class="dropZone__commnentCount">
      All done! <span>{commentCount}</span> comments created :)
    </div>
    <img src="/assets/check.svg" class="dropZone__image" alt="" />
    <Button
      style="transform: tranlateX(-1em)"
      on:click={() => (downloadDone = false)}
      variant="raised">Commentify again</Button
    >
  {/if}
</div>

<style>
  .dropZone {
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    width: 50em;
    aspect-ratio: 1.6;
    outline: 0.2em dashed var(--color-secondary);
    outline-offset: -1em;
    background-color: var(--color-white);
    transition: background-color 300ms ease-in-out;
  }

  .dropZone__introText {
    margin-bottom: 2em;
  }

  .dropZone.dragOver {
    background-color: var(--color-secondary-light);
  }

  .dropZone__commnentCount {
    margin-bottom: 1em;
  }

  .dropZone__commnentCount span {
    font-weight: bold;
  }

  .dropZone__image {
    width: 10em;
    margin-bottom: 1em;
    height: min-content;
  }

  .dropZone__childNodes {
    margin-bottom: 2em;
  }
</style>
