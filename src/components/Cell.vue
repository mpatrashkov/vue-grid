<template>
    <drag
        :data="cellId"
        class="cell"
        :style="{
            'background-color': color,
        }"
    >
        <div class="drop-zones">
            <drop
                v-for="zone in zones"
                :key="zone"
                @drop="drop($event, zone)"
                :class="zone"
                :accepts-data="(id) => id !== cellId"
            />
        </div>

        <button @click="remove" class="removeBtn">kill</button>
    </drag>
</template>

<script>
import { Drag, Drop } from "vue-easy-dnd"
export default {
    name: "Cell",
    data: () => ({
        zones: ["top", "bottom", "left", "right", "replace"],
    }),
    props: {
        cellId: Number,
        color: String,
    },
    methods: {
        drop(e, zone) {
            // Bad practice, move to Vuex
            this.$root.$emit("move", e.data, this.cellId, zone)
        },
        remove() {
            this.$root.$emit("remove", this.cellId)
        },
    },
    components: {
        Drag,
        Drop,
    },
}
</script>

<style scoped>
.cell {
    width: 100%;
    height: 100%;
    position: relative;
}

.drop-zones > * {
    position: absolute;
    border: 2px dotted gray;
}

.top {
    top: 0;
    left: 25%;
    right: 25%;
    height: 25%;
}

.bottom {
    bottom: 0;
    left: 25%;
    right: 25%;
    height: 25%;
}

.left {
    top: 0;
    left: 0;
    width: 25%;
    bottom: 0;
}

.right {
    top: 0;
    right: 0;
    width: 25%;
    bottom: 0;
}

.replace {
    top: 25%;
    left: 25%;
    right: 25%;
    bottom: 25%;
}

.removeBtn {
    position: absolute;
    z-index: 2;
}
</style>
