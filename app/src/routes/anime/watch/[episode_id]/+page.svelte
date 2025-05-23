<script lang="ts">
  import { tick } from "svelte";
  import type { EpisodeIdSlug } from "./proxy+page";
  import { FetchAnimeApi } from "../../../../utils/Fetch";
  import WatchAnimeLayout from "../../../../components/layouts/WatchAnimeLayout.svelte";
  import LoadingElements from "../../../../components/elements/LoadingElements.svelte";

  export let data: EpisodeIdSlug;
  let episodeData: any = null;
  let videoUrl: string | null = null;
  let notFound = false;
  let isLoading = true;
  let title: string;

  async function fetchEpisode() {
    try {
      const response = await FetchAnimeApi.get(`/episode/${data.episodeId}`);

      if (response.status === 404) {
        notFound = true;
      } else {
        episodeData = response.data.data;
        title = episodeData.title;

        const firstServer = episodeData.server.qualities?.find(
          (q: any) => q.serverList.length > 0,
        )?.serverList[0];

        if (firstServer) {
          await fetchVideoUrl(firstServer.serverId);
        } else {
          videoUrl = episodeData?.defaultStreamingUrl;
        }
      }
    } catch (error) {
      console.error("Failed to fetch episode:", error);
      notFound = true;
    } finally {
      isLoading = false;
    }
  }

  async function fetchVideoUrl(serverId: string) {
    try {
      const serverResponse = await FetchAnimeApi.get(`/server/${serverId}`);
      const fetchedUrl =
        serverResponse.data.data.url || episodeData?.defaultStreamingUrl;

      await checkVideo(fetchedUrl);
    } catch (error) {
      console.error("Failed to fetch video URL:", error);
      videoUrl = episodeData?.defaultStreamingUrl;
      await tick();
    }
  }

  async function checkVideo(url: string) {
    try {
      const res = await fetch(url, { method: "HEAD" });

      if (res.ok) {
        videoUrl = url;
      } else {
        videoUrl = episodeData?.defaultStreamingUrl;
      }
      await tick();
    } catch (e) {
      console.error("Failed to fetch video:", e);
      videoUrl = episodeData?.defaultStreamingUrl;
      await tick();
    }
  }

  fetchEpisode();
</script>

<svelte:head>
  <title>ComicHive - {title}</title>
</svelte:head>

<div>
  {#if isLoading}
    <LoadingElements />
  {:else if notFound}
    <div class="min-h-screen flex justify-center items-center">
      <h1 class="text-[hsl(var(--background-foreground))] font-bold text-1xl">
        Mohon maaf, episode Id anda salah.
      </h1>
    </div>
  {:else}
    <WatchAnimeLayout
      episode={episodeData}
      {videoUrl}
      episodeId={data.episodeId}
    />
  {/if}
</div>
