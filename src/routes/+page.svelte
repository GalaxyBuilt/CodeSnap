<script lang="ts">
  import { onMount } from "svelte";
  import { listen } from "@tauri-apps/api/event";
  import { Search, Plus, Folder, Tag, Star, Clipboard, Trash2, Settings, Code as CodeIcon } from "lucide-svelte";

  // State
  let snippets = $state([
    { id: 1, title: "Example Python Snippet", code: "print('Hello, CodeSnap!')", language: "python" },
    { id: 2, title: "Svelte Greeting", code: "console.log('Hello from Svelte!')", language: "javascript" }
  ]);
  let searchQuery = $state("");
  let selectedSnippetId = $state(1);

  // Computed
  let filteredSnippets = $derived(
    snippets.filter(s => 
      s.title.toLowerCase().includes(searchQuery.toLowerCase()) || 
      s.code.toLowerCase().includes(searchQuery.toLowerCase())
    )
  );
  
  let selectedSnippet = $derived(snippets.find(s => s.id === selectedSnippetId));

  onMount(async () => {
    // Listen for captured snippets from the backend
    const unlisten = await listen<string>("snippet-captured", (event) => {
      const newSnippet = {
        id: Date.now(),
        title: "New Captured Snippet",
        code: event.payload,
        language: "text" // Placeholder, AI will detect this
      };
      snippets = [newSnippet, ...snippets];
      selectedSnippetId = newSnippet.id;
    });

    return () => {
      unlisten();
    };
  });

  function copyToClipboard(code: string) {
    navigator.clipboard.writeText(code);
    // Add toast notification later
  }
</script>

<div class="flex h-screen bg-zinc-950 text-zinc-100 font-sans overflow-hidden">
  <!-- Sidebar -->
  <aside class="w-64 border-r border-zinc-800 flex flex-col bg-zinc-900/50 backdrop-blur-xl">
    <div class="p-4 border-b border-zinc-800 flex items-center justify-between">
      <h1 class="font-bold text-lg flex items-center gap-2">
        <CodeIcon class="w-5 h-5 text-blue-500" />
        CodeSnap
      </h1>
      <button class="p-1 hover:bg-zinc-800 rounded transition-colors text-zinc-400 hover:text-white">
        <Settings class="w-4 h-4" />
      </button>
    </div>

    <nav class="flex-1 overflow-y-auto p-2 space-y-4">
      <div>
        <h2 class="px-3 text-xs font-semibold text-zinc-500 uppercase tracking-wider mb-2">Library</h2>
        <ul class="space-y-1 text-sm">
          <li>
            <button class="w-full text-left px-3 py-2 rounded bg-blue-600/10 text-blue-400 flex items-center gap-2">
              <Clipboard class="w-4 h-4" /> All Snippets
            </button>
          </li>
          <li>
            <button class="w-full text-left px-3 py-2 rounded hover:bg-zinc-800 text-zinc-400 flex items-center gap-2 transition-colors">
              <Star class="w-4 h-4" /> Favorites
            </button>
          </li>
        </ul>
      </div>

      <div>
        <div class="px-3 flex items-center justify-between mb-2">
          <h2 class="text-xs font-semibold text-zinc-500 uppercase tracking-wider">Folders</h2>
          <button class="text-zinc-500 hover:text-white transition-colors"><Plus class="w-3 h-3" /></button>
        </div>
        <ul class="space-y-1 text-sm text-zinc-400">
          <li>
            <button class="w-full text-left px-3 py-1.5 rounded hover:bg-zinc-800 flex items-center gap-2 transition-colors">
              <Folder class="w-4 h-4" /> Python
            </button>
          </li>
          <li>
            <button class="w-full text-left px-3 py-1.5 rounded hover:bg-zinc-800 flex items-center gap-2 transition-colors">
              <Folder class="w-4 h-4" /> Frontend
            </button>
          </li>
        </ul>
      </div>

      <div>
        <div class="px-3 flex items-center justify-between mb-2">
          <h2 class="text-xs font-semibold text-zinc-500 uppercase tracking-wider">Tags</h2>
          <button class="text-zinc-500 hover:text-white transition-colors"><Plus class="w-3 h-3" /></button>
        </div>
        <div class="px-3 flex flex-wrap gap-2">
          <span class="px-2 py-0.5 bg-zinc-800 border border-zinc-700 rounded text-xs text-zinc-400">#react</span>
          <span class="px-2 py-0.5 bg-zinc-800 border border-zinc-700 rounded text-xs text-zinc-400">#api</span>
          <span class="px-2 py-0.5 bg-zinc-800 border border-zinc-700 rounded text-xs text-zinc-400">#auth</span>
        </div>
      </div>
    </nav>
  </aside>

  <!-- List -->
  <main class="w-80 border-r border-zinc-800 flex flex-col bg-zinc-950">
    <div class="p-4 border-b border-zinc-800">
      <div class="relative group">
        <Search class="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-zinc-500 group-focus-within:text-blue-500 transition-colors" />
        <input 
          type="text" 
          placeholder="Search snippets..." 
          bind:value={searchQuery}
          class="w-full pl-9 pr-4 py-2 bg-zinc-900 border border-zinc-800 rounded-lg text-sm focus:outline-none focus:border-blue-500 transition-all placeholder:text-zinc-600 focus:ring-1 focus:ring-blue-500/20"
        />
      </div>
    </div>

    <div class="flex-1 overflow-y-auto">
      {#each filteredSnippets as snippet}
        <button 
          onclick={() => selectedSnippetId = snippet.id}
          class="w-full text-left p-4 border-b border-zinc-900/50 hover:bg-white/[0.02] transition-colors relative group {selectedSnippetId === snippet.id ? 'bg-blue-600/5 border-l-2 border-l-blue-500' : ''}"
        >
          <div class="flex justify-between items-start mb-1">
            <h3 class="font-medium text-sm truncate pr-2 {selectedSnippetId === snippet.id ? 'text-blue-400' : 'text-zinc-200'}">{snippet.title}</h3>
            <span class="text-[10px] px-1.5 py-0.5 bg-zinc-800 border border-zinc-700 rounded text-zinc-500 uppercase font-bold tracking-tighter">{snippet.language}</span>
          </div>
          <p class="text-xs text-zinc-500 line-clamp-2 font-mono opacity-80">{snippet.code.slice(0, 100)}</p>
        </button>
      {/each}
    </div>
  </main>

  <!-- Content -->
  <section class="flex-1 flex flex-col bg-[#0d1117]">
    {#if selectedSnippet}
      <div class="p-4 border-b border-zinc-800 flex items-center justify-between bg-zinc-950/50 backdrop-blur-sm">
        <div class="flex items-center gap-3">
          <input 
            type="text" 
            bind:value={selectedSnippet.title}
            class="bg-transparent border-none focus:ring-0 text-lg font-bold text-white w-96 p-0"
          />
        </div>
        <div class="flex items-center gap-2">
          <button 
            onclick={() => copyToClipboard(selectedSnippet.code)}
            class="px-3 py-1.5 bg-zinc-900 hover:bg-zinc-800 border border-zinc-700 rounded-lg text-xs font-medium flex items-center gap-2 transition-all active:scale-95"
          >
            <Clipboard class="w-3.5 h-3.5" /> Copy Code
          </button>
          <button class="p-1.5 hover:bg-red-500/10 hover:text-red-400 rounded-lg text-zinc-500 transition-colors">
            <Trash2 class="w-4 h-4" />
          </button>
        </div>
      </div>
      <div class="flex-1 p-6 overflow-auto font-mono text-sm leading-relaxed">
        <pre class="w-full bg-zinc-900/30 p-4 rounded-xl border border-zinc-800/50 overflow-x-auto">
          <code>{selectedSnippet.code}</code>
        </pre>
      </div>
    {:else}
      <div class="flex-1 flex flex-col items-center justify-center text-zinc-500">
        <CodeIcon class="w-12 h-12 mb-4 opacity-20" />
        <p>Select a snippet to view code</p>
      </div>
    {/if}
  </section>
</div>

<style>
  :global(body) {
    margin: 0;
    overflow: hidden;
  }
</style>
