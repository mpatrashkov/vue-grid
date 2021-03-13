<template>
    <div
        class="container"
        :class="{
            vertical: vertical,
            horizontal: horizontal,
        }"
    >
        <grid-elelement :data="first" />
        <splitter :orientation="orientation" @resize="resize" />
        <grid-elelement :data="second" />
    </div>
</template>

<script>
import Splitter from "./Splitter.vue"

export default {
    name: "Container",
    props: {
        orientation: String,
        first: Object,
        second: Object,
    },
    computed: {
        horizontal() {
            return this.orientation === "horizontal"
        },
        vertical() {
            return this.orientation === "vertical"
        },
    },
    methods: {
        resize(sizeChange) {
            const splitterWidth = 5

            const containerSizeInPx =
                (this.horizontal
                    ? this.$el.offsetWidth
                    : this.$el.offsetHeight) - splitterWidth

            const firstCellSize = this.first.size
            const firstCellSizeInPx = (firstCellSize * containerSizeInPx) / 100

            const firstCellNewSize =
                ((firstCellSizeInPx + sizeChange) / containerSizeInPx) * 100

            this.first.size = firstCellNewSize
            this.second.size += firstCellSize - firstCellNewSize
        },
    },
    components: {
        Splitter,
        GridElelement: () => import("./GridElelement.vue"),
    },
}
</script>

<style scoped>
.container {
    width: 100%;
    height: 100%;

    flex: 1;
    display: flex;
}

.vertical {
    flex-direction: column;
}

.horizontal {
    flex-direction: row;
}
</style>
