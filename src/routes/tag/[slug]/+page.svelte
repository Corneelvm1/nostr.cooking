<script lang="ts">
  import { page } from '$app/stores';
  import { ndk } from '$lib/nostr';
  import type { NDKEvent, NDKFilter } from '@nostr-dev-kit/ndk';
  import { onMount } from 'svelte';
  import Feed from '../../../components/Feed.svelte';
  import { validateMarkdownTemplate } from '$lib/pharser';
  import { recipeTags } from '$lib/consts';
  import { goto } from '$app/navigation';
  import TagsSearchAutocomplete from '../../../components/TagsSearchAutocomplete.svelte';

  let tag: string | undefined = undefined;
  let events: NDKEvent[] = [];
  let loaded = false;

  $: {
    if ($page.params.slug) {
      loadData();
    }
  }

  function openTag(query: string) {
    goto(`/tag/${query}`);
  }

  async function loadData() {
    loaded = false;
    events = [];
    let filter: NDKFilter = {
      limit: 256,
      kinds: [30023],
      '#t': [`nostrcooking-${$page.params.slug.toLowerCase().replaceAll(' ', '-')}`]
    };
    const evts = await $ndk.fetchEvents(filter);
    let evtsArr = Array.from(evts);
    evtsArr.forEach((ev, i) => {
      if (validateMarkdownTemplate(ev.content) == null) {
        evtsArr.splice(i, 1);
      }
    });
    events = evtsArr;
    loaded = true;
  }
</script>

<TagsSearchAutocomplete
  placeholderString={"Look for a specific tag, like 'italian', 'steak' or 'glutenfree'"}
  actionString={'Go'}
  action={openTag}
/>

<div class="prose mb-6">
  <h1>
    Recipes with the tag "{#if $page.params.slug}{#if recipeTags.find((e) => e.title
            .toLowerCase()
            .replaceAll(' ', '-') == $page.params.slug
            ?.toLowerCase()
            .replaceAll(' ', '-'))}{recipeTags.find(
          (e) =>
            e.title.toLowerCase().replaceAll(' ', '-') ==
            $page.params.slug?.toLowerCase().replaceAll(' ', '-')
        )?.emoji
          ? `${
              recipeTags.find(
                (e) =>
                  e.title.toLowerCase().replaceAll(' ', '-') ==
                  $page.params.slug?.toLowerCase().replaceAll(' ', '-')
              )?.emoji
            } ${
              recipeTags.find(
                (e) =>
                  e.title.toLowerCase().replaceAll(' ', '-') ==
                  $page.params.slug?.toLowerCase().replaceAll(' ', '-')
              )?.title
            }`
          : `${
              recipeTags.find(
                (e) =>
                  e.title.toLowerCase().replaceAll(' ', '-') ==
                  $page.params.slug?.toLowerCase().replaceAll(' ', '-')
              )?.title
            }`}{:else}{$page.params.slug}{/if}{:else}...{/if}"
  </h1>
</div>

{#if events.length > 0}
  <Feed {events} />
{:else if loaded == false}
  <p>loading</p>
{:else}
  <p>Nothing found here :(</p>
{/if}
