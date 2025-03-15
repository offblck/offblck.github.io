<script lang="ts">
    import { tick } from "svelte";

    const categories = [
        "projects",
        "blog",
        "oomfs",
        "twitter",
        "favorites",
        //"effects",
        //"themes",
    ];
    let filteredCategories = $state(categories);
    let selected = $state(0);
    let isFzf = $state(true);
    let fzfInput = $state("");
    let promptValue = $state("");
    let paused = $state(false);
    let completion: string | undefined = $derived.by(() => {
        if (promptValue) {
            return categories.find((category) =>
                category.startsWith(promptValue),
            );
        } else {
            return undefined;
        }
    });
    let promptLines: string[] = $state([]);

    let text_div: HTMLDivElement | undefined = $state();

    function lockFocus(e: any) {
        e.target.focus();
    }

    function clearPrompts() {
        promptLines = [];
        promptValue = "";
    }

    function setCursorToEnd() {
        let selectedText = window.getSelection();
        if (selectedText) {
            let selectedRange = document.createRange();
            if (text_div) {
                if (text_div.firstChild) {
                    selectedRange.setStart(
                        text_div.firstChild,
                        promptValue.length,
                    );
                }
                selectedRange.collapse(true);
                selectedText.removeAllRanges();
                selectedText.addRange(selectedRange);
                text_div.focus();
            }
        }
    }

    function hasModifier(e: KeyboardEvent) {
        return e.altKey || e.ctrlKey || e.shiftKey || e.metaKey;
    }

    function handlePromptKeydown(e: any) {
        if (e.code === "Enter") {
            e.preventDefault();
            if (categories.includes(e.target.textContent)) {
                promptLines.push(e.target.textContent);
                promptValue = "";
            } else if (promptValue.includes("clear")) {
                clearPrompts();
            } else {
                promptLines.push;
            }
        } else if (e.key === "l" && e.ctrlKey) {
            clearPrompts();
        } else if (e.key === "u" && e.ctrlKey) {
            promptValue = "";
        } else if (e.key === "Tab") {
            e.preventDefault();
            if (completion) {
                promptValue = completion;
                tick().then(setCursorToEnd);
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

    function handleInput(input: any) {
        handlePause();
        fuzzySearch(input);
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
    <div class="flex flex-col md:flex-row gap-1 items-start md:items-end">
        <pre
            class="text-2xs xxs:text-xs xs:text-sm leading-tight select-none text-light-green">
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
    <div class="my-6 flex gap-6 items-center ml-2 xxs:ml-8">
        <img
            class="rounded-full w-16 h-16 border-2 border-red-600"
            src="/feitan.jpeg"
            alt="feitan pfp"
        />
        Hello I'm OFFBLACK an absolute random with internet access
    </div>
    <div class="ml-[36px] xxs:ml-[60px] flex flex-col gap-4">
        <div class="flex gap-6">
            <div class="relative">
                <div
                    style:height={categories.length * 38 + "px"}
                    class="bg-dark-gray w-2"
                ></div>
                {#if filteredCategories.length}
                    <div
                        class="absolute left-0 w-[124px] bg-light-gray h-6"
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
                        {category}
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
                    oninput={() => handleInput(fzfInput)}
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
        {#if !isFzf}
            {#each promptLines as promptLine}
                <div class="flex gap-2">
                    {@render prompt()}
                    <span>{promptLine}</span>
                </div>
                {#if promptLine === "projects"}
                    shiit, you supposed to ship?
                {:else if promptLine === "blog"}
                    you think I have opinions?
                {:else if promptLine === "oomfs"}
                    could be you but you playing...
                {:else if promptLine === "twitter"}
                    <a
                        class="w-max"
                        href="https://x.com/OFFBLACKdev"
                        target="_blank">https://x.com/OFFBLACKdev</a
                    >
                {:else if promptLine === "favorites"}
                    kitty, nvim, c, rust
                {/if}
            {/each}
            <div class="flex">
                <div class="pr-2">
                    {@render prompt()}
                </div>
                <div
                    class="focus:outline-none active:border-none inline caret-transparent
                    w-fit tracking-wide {completion
                        ? 'text-white'
                        : 'text-red-500'}"
                    bind:textContent={promptValue}
                    autofocus
                    contenteditable="true"
                    role="form"
                    onkeydown={handlePromptKeydown}
                    onblur={lockFocus}
                    bind:this={text_div}
                ></div>
                <div>
                    <span
                        class="bg-white absolute w-[1ch] {paused
                            ? 'animate-none'
                            : 'animate-blink'}"
                        >L
                    </span>
                    {#if completion}
                        <span class="text-gray-500">
                            {completion.slice(
                                promptValue.length,
                                promptValue.length + 1,
                            )}
                        </span>
                    {/if}
                </div>

                {#if promptValue !== "" && completion}
                    <div class="text-gray-500">
                        {completion.slice(promptValue.length + 1)}
                    </div>
                {/if}
            </div>
        {/if}
    </div>
</main>
