<script lang="ts">
  import FileHandler from "./components/FileHandler.svelte";
  import TopBar from "./components/TopBar.svelte";
</script>

<svelte:head>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" />
  <link
    href="https://fonts.googleapis.com/css2?family=Roboto&display=swap"
    rel="stylesheet"
  />
</svelte:head>

<main>
  <TopBar />

  <div class="content">
    <p class="intro">
      HTML_Commentify puts the classnames of your HTML-elements in a comment
      after the closing tag. This makes life easier when working with larger
      HTML files. Just drop your HTML-files and behold the magic! &#128512
    </p>
    <FileHandler />
  </div>
</main>

<style>
  :global(:root) {
    --color-text: #000000;
    --color-primary: #14213d;
    --color-secondary: #fca311;
    --color-secondary-light: #fca21175;
    --color-gray: #e5e5e5;
    --color-white: #ffffff;

    --font-primary: "Roboto", sans-serif;
  }

  :global(body) {
    background-color: var(--color-gray);
  }

  main,
  .content {
    display: flex;
    flex-flow: column;
    height: 100%;
  }

  .content {
    position: relative;
    justify-content: center;
    align-items: center;
    padding: 1em;
  }

  .intro {
    max-width: 40em;
    text-align: center;
    margin-bottom: 3em;
    line-height: 1.4;
  }
</style>
