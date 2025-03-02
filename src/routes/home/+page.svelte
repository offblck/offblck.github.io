<script lang="ts">
    const categories = [
        "projects",
        "blog",
        "oomfs",
        "twitter",
        "favorite",
        //"effects",
        //"themes",
    ];
    let filteredCategories = $state(categories);
    let selected = $state(0);
    let isFzf = $state(true);
    let fzfInput = $state("");
    let promptValue = $state("");
    let paused = $state(false);
    let promptLines: string[] = $state([]);
    function lockFocus(e: any) {
        e.target.focus();
    }
    function handlePromptKeydown(e: any) {
        if (e.code === "Enter") {
            e.preventDefault();
            if (categories.includes(e.target.textContent)) {
                promptLines.push(e.target.textContent);
                promptValue = "";
            }
        }
    }

    function handleKeydown(e: any) {
        switch (e.code) {
            case "Enter":
                if (selected < filteredCategories.length) {
                    handleEntry(filteredCategories[selected]);
                }
                e.preventDefault();
                break;
            case "Tab":
                e.preventDefault();
                if (e.shiftKey) {
                    selected = Math.min(
                        filteredCategories.length - 1,
                        selected + 1,
                    );
                } else {
                    selected = Math.max(0, selected - 1);
                }
                break;
            case "ArrowUp":
                selected = Math.min(
                    filteredCategories.length - 1,
                    selected + 1,
                );
                break;
            case "ArrowDown":
                selected = Math.max(0, selected - 1);
                break;
        }
    }
    function handleEntry(text: string) {
        if (isFzf) {
            isFzf = false;
        }
        promptLines.push(text);
        filteredCategories = categories;
    }
    function handleInput() {
        handlePause();
        fuzzySearch(fzfInput);
    }
    function fuzzySearch(input: string) {
        if (!input || input.trim() === "") {
            filteredCategories = [...categories];
            return;
        }

        const searchTerm = input.toLowerCase();

        const matches = categories.map((category) => {
            const score = fuzzyMatchScore(category, searchTerm);
            return { category, score };
        });

        const result = matches
            .filter((match) => match.score > 0)
            .sort((a, b) => b.score - a.score)
            .map((match) => match.category);

        filteredCategories = result;
    }

    function fuzzyMatchScore(str: string, pattern: string) {
        if (!pattern) return 1;
        if (!str) return 0;

        let patternIdx = 0;
        let strIdx = 0;
        let matches = [];

        while (patternIdx < pattern.length && strIdx < str.length) {
            if (pattern[patternIdx] === str[strIdx]) {
                matches.push(strIdx);
                patternIdx++;
            }
            strIdx++;
        }

        if (patternIdx < pattern.length) return 0;

        let score = 100;

        let consecutiveCount = 0;
        for (let i = 1; i < matches.length; i++) {
            if (matches[i] === matches[i - 1] + 1) {
                consecutiveCount++;
                score += 15 * consecutiveCount;
            } else {
                consecutiveCount = 0;
            }
        }

        for (let i = 0; i < matches.length; i++) {
            const pos = matches[i];
            if (pos === 0 || !isAlphaNumeric(str[pos - 1])) {
                score += 25;
            }
        }

        let totalGap =
            matches[matches.length - 1] - matches[0] - (matches.length - 1);
        score -= totalGap * 0.5;

        score -= matches[0] * 0.5;

        score = Math.max(0, Math.min(100, score));

        score *= pattern.length / Math.max(pattern.length, str.length);

        return score;
    }

    function isAlphaNumeric(char: string) {
        return /[a-zA-Z0-9]/.test(char);
    }
    function handlePause() {
        paused = true;
        setTimeout(() => (paused = false), 300);
    }
</script>

{#snippet prompt()}
    <pre class="font-jetbrains">[<span class="text-light-blue">guest</span
        >@<span class="text-light-green">offblack.dev</span>]<span
            class="text-red-600">$</span
        ></pre>
{/snippet}

<main class="font-jetbrains text-white p-4 pb-16">
    {@render prompt()}
    <div class="flex gap-1 items-end">
        <pre class="text-sm leading-tight select-none text-light-green">
   ____  ________________  __    ___   ________ __
  / __ \/ ____/ ____/ __ )/ /   /   | / ____/ //_/
 / / / / /_  / /_  / __  / /   / /| |/ /   / , &lt;  
/ /_/ / __/ / __/ / /_/ / /___/ ___ / /___/ /| |  
\____/_/   /_/   /_____/_____/_/  |_\____/_/ |_|  
</pre>
        <span class="text-sm">4F 46 46 42 4C 41 43 4B</span>
    </div>
    <div class="flex items-center gap-2 mt-4">
        {@render prompt()}
        about
    </div>
    <div class="my-6 flex gap-6 items-center ml-8">
        <img
            class="rounded-full w-16 h-16 border-2 border-red-600"
            src="/feitan.jpeg"
            alt="feitan pfp"
        />
        Hello I'm OFFBLACK an absolute random with internet access
    </div>
    <div class="ml-[60px] flex flex-col gap-4">
        <div class="flex gap-6">
            <div class="relative">
                <div
                    style:height={categories.length * 38 + "px"}
                    class="bg-dark-gray w-2"
                ></div>
                {#if filteredCategories.length}
                    <div
                        class="absolute left-0 w-[112px] bg-light-gray h-6"
                        style:top={(categories.length - 1 - selected) * 41 +
                            "px"}
                    >
                        <div class="w-2 h-full bg-red-600"></div>
                    </div>
                {/if}
            </div>
            <ul class="flex flex-col gap-4 relative justify-end">
                {#each filteredCategories as category}
                    <li class="flex gap-2">
                        {#if category === "favorite"}
                            {category}
                            <span class="text-gray-400"
                                >[tech | music | languages | books]</span
                            >
                        {:else}
                            {category}
                        {/if}
                    </li>
                {/each}
            </ul>
        </div>
        <span
            class={filteredCategories.length
                ? "text-dark-green"
                : "text-red-600"}
            >{filteredCategories.length}/{categories.length}</span
        >
        {#if isFzf}
            <div class="flex gap-1">
                <span class="block text-light-blue mr-1">&gt;</span>
                <div
                    class="focus:outline-none active:border-none inline caret-transparent
            w-fit tracking-wide"
                    autofocus
                    bind:textContent={fzfInput}
                    contenteditable="true"
                    role="form"
                    onkeydown={handleKeydown}
                    oninput={handleInput}
                    onblur={lockFocus}
                ></div>
                <span
                    class="bg-white w-[1ch] {paused
                        ? 'animate-none'
                        : 'animate-blink'}"
                ></span>
            </div>
        {:else}
            <div>&nbsp;</div>
        {/if}
    </div>
    <div class="flex flex-col gap-2">
        {#if promptLines.length}
            {#each promptLines as promptLine}
                <div class="flex gap-2">
                    {@render prompt()}
                    <span>{promptLine}</span>
                </div>
                {#if promptLine === "projects"}
                    shit, you supposed to ship?
                {:else if promptLine === "blog"}
                    you think I have opinions?
                {:else if promptLine === "oomfs"}
                    <a href="https://x.com/pokeghosst" target="_blank"
                        >pokeghost</a
                    >
                {:else if promptLine === "twitter"}
                    <a href="https://x.com/OFFBLACKdev" target="_blank"
                        >https://x.com/OFFBLACKdev</a
                    >
                {:else if promptLine === "favorite"}
                    Soon TM
                {/if}
            {/each}
            <div class="flex gap-[6px]">
                {@render prompt()}
                <div
                    class="focus:outline-none active:border-none inline caret-transparent
                    w-fit tracking-wide"
                    bind:textContent={promptValue}
                    autofocus
                    contenteditable="true"
                    role="form"
                    onkeydown={handlePromptKeydown}
                    onblur={lockFocus}
                ></div>
                <span
                    class="bg-white w-[1ch] {paused
                        ? 'animate-none'
                        : 'animate-blink'}"
                ></span>
            </div>
        {/if}
    </div>
</main>
