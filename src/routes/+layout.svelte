<script lang="ts">
    import '../syles/app.scss'
</script>

<svelte:head>
    <title>SvelteTyper</title>
</svelte:head>

<div class="layout">
    <nav>
        <span style="display:flex"><h1>🔥 SvelteTyper</h1><h7 style="opacity: 0.4; padding-left:0.6rem;">by Eugene Reim</h7></span>
    </nav>

    <main>
        <slot />
    </main>
</div>

<style lang="scss">
    h1 {
        font-family: 'Lexend Deca', sans-serif;
        font-size: 1.8rem;
        color: var(--fg-200);
        letter-spacing: 2px;
    }

    .layout {
        height: 100%;
        display: grid;
        grid-template-rows: auto 1fr;
        align-items: center;
        padding: 2rem;
    }
</style>