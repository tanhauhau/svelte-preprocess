<script lang="ts" context="module">
    import { storeModuleTemplateOnly, storeModuleScriptOnly } from "./store";
    onlyUsedInModuleScript;
</script>

<script lang="ts">
    // Some of these imports don't exist on disk, but they don't need to for this test.
    import { fly } from "svelte/transition";
    import { flip } from "svelte/animate";
    import Nested from "./Nested.svelte";
    import { hello } from "./script";
    import { AValue, AType } from "./types";
    import { storeTemplateOnly, storeScriptOnly } from "./store";
    import { onlyUsedInModuleScript } from "./modulescript";
    const ui = { MyNested: Nested };
    const val: AType = "test1";
    const prom: Promise<AType> = Promise.resolve("test2");
    const arr: AType[] = ["test1", "test2"];
    const isTest1 = (v: AType) => v === "test1";
    const obj = {
        fn: () => "test",
        val: "test1" as const,
        $in: "",
        a: "$",
        b: '$',
        c: `$`
    };
    let inputVal: string;
    const action = (node: Element, options: { id: string; }) => { node.id = options.id; };
    const action2 = (node: Element) => { node.classList.add("test"); };
    let nested: Nested;
    
    let scrollY = 500;
    let innerWidth = 500;
    
    const duration = 200;
    function onKeyDown(e: KeyboardEvent): void {
        e.preventDefault();
    }
    $storeScriptOnly;
    $storeModuleScriptOnly;

    // These shouldn't count as store values:
    // $\\; $$; $§; $%; $°; $(; $); $[; $]; $<; $>; $ ; $^; $`; $"; $';
    // These don't need to be tested explicitly, the test will throw and error
    // at the TypeScript parsing stage if they are part of the false positives
</script>

<style>
    .value { color: #ccc; }
</style>

<svelte:window on:keydown={onKeyDown} {scrollY} bind:innerWidth />
<svelte:body on:keydown={onKeyDown} />

<svelte:head>
    <title>Title: {val}</title>
</svelte:head>

{$storeTemplateOnly}
{$storeModuleTemplateOnly}

<div>
    <Nested let:var1 let:var2={var3}>
        <Nested bind:this={nested} />
        <Nested {...obj} />
        <Nested {...{ var1, var3 }} />

        <svelte:fragment slot="slot1" let:var4={var5}>
            <Nested {...{ var5 }} />
        </svelte:fragment>

        <div slot="slot2" let:var6={var7}>
            <Nested {...{ var7 }} />
        </div>

        <div slot="slot3">
            <Nested {...{ val }} />
        </div>
    </Nested>

    <svelte:component this={ui.MyNested} {val} on:keydown={onKeyDown} bind:inputVal />

    <p class:value={!!inputVal}>{hello}</p>
    <input bind:value={inputVal} use:action={{ id: val }} use:action2 />

    {#if AValue && val}
        <p class="value" transition:fly={{ duration }}>There is a value: {AValue}</p>
    {/if}

    {#if val && isTest1(val) && AValue && true && "test"}
        <p class="value">There is a value: {AValue}</p>
    {:else if obj.val && obj.fn() && isTest1(obj.val)}
        <p class="value">There is a value: {AValue}</p>
    {:else}
        Else
    {/if}

    {#each arr as item (item)}
        <p animate:flip={{ duration }}>{item}</p>
    {/each}

    {#each arr as item}
        <p>{item}</p>
    {:else}
        <p>No items</p>
    {/each}

    {#await prom}
        Loading...
    {:then value} 
        <input type={val} {value} on:input={e => inputVal = e.currentTarget.value} />
    {:catch err}
        <p>Error: {err}</p>
    {/await}

    {#await prom then value}
        <p>{value}</p>
    {/await}

    {#key val}
        <p>Keyed {val}</p>
    {/key}

    <slot name="slot0" {inputVal}>
        <p>{inputVal}</p>
    </slot>

    {@html val}
    {@debug val, inputVal}
</div>

<!-- Some comment -->
