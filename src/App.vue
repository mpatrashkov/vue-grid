<template>
    <div id="app">
        <grid />
    </div>
</template>

<script>
import Grid from "./components/Grid.vue"

export default {
    name: "App",
    components: {
        Grid,
    },
}
</script>

<style>
body,
html {
    width: 100%;
    height: 100%;
    padding: 0;
    margin: 0;
}

#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #2c3e50;

    width: 100%;
    height: 100%;
}
</style>
