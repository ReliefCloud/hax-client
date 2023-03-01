<script lang="ts">
  import {filter, identity} from 'ramda'
  import Input from "src/partials/Input.svelte"
  import Modal from "src/partials/Modal.svelte"
  import Content from "src/partials/Content.svelte"
  import Anchor from "src/partials/Anchor.svelte"
  import {listenForFile, stripExifData, blobToFile} from "src/util/html"
  import {uploadFile, postJson} from "src/util/misc"
  import user from "src/agent/user"

  export let value
  export let icon
  export let maxWidth = null
  export let maxHeight = null

  let input, file, listener, quote
  let loading = false
  let isOpen = false

  $: {
    if (input) {
      listener = listenForFile(input, async inputFile => {
        const opts = filter(identity, {maxWidth, maxHeight})

        file = blobToFile(await stripExifData(inputFile, opts))
        quote = await postJson(user.dufflepud('/upload/quote'), {
          uploads: [{size: file.size}],
        })
      })
    }
  }

  const accept = async () => {
    loading = true

    try {
      const {id} = quote.uploads[0]
      const {url} = await uploadFile(user.dufflepud(`/upload/${id}`), file)

      value = url
    } finally {
      loading = false
    }

    file = null
    quote = null
    isOpen = false
  }

  const decline = () => {
    file = null
    quote = null
    isOpen = false
  }
</script>

<div class="flex gap-2">
  <Input type="text" wrapperClass="flex-grow" bind:value={value} placeholder="https://">
    <i slot="before" class={`fa fa-${icon}`} />
  </Input>
  <Anchor type="button" on:click={() => { isOpen = true }}>
    <i class="fa fa-upload" />
  </Anchor>
</div>

{#if quote}
<Modal onEscape={decline}>
  <Content>
    <h1 class="staatliches text-2xl">Confirm File Upload</h1>
    <p>Please accept the following terms:</p>
    <p>{quote.terms}</p>
    <div class="flex gap-2">
      <Anchor type="button" on:click={decline} {loading}>Decline</Anchor>
      <Anchor type="button-accent" on:click={accept} {loading}>Accept</Anchor>
    </div>
  </Content>
</Modal>
{:else if isOpen}
<Modal onEscape={decline}>
  <Content>
    <h1 class="staatliches text-2xl">Upload a File</h1>
    <p>Click below to select a file to upload.</p>
    <input type="file" bind:this={input} />
  </Content>
</Modal>
{/if}