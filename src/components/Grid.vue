<template>
    <div class="grid">
        <grid-elelement
            v-if="gridData"
            :data="{ size: 1, el: gridData }"
            :index="0"
        />
        <div class="button">
            <button @click="add">Add</button>
        </div>

        <div class="drop-zones">
            <drop
                v-for="zone in dropZones"
                :key="zone"
                @drop="drop($event, zone)"
                :class="zone"
            />
        </div>
    </div>
</template>

<script>
import { Drop } from "vue-easy-dnd"
import GridElelement from "./GridElelement.vue"

export default {
    components: { Drop, GridElelement },
    name: "Grid",
    data: () => ({
        dropZones: ["top", "bottom", "left", "right"],
        gridData: {
            id: 0,
            type: "container",
            orientation: "horizontal",
            first: {
                size: 40,
                el: {
                    id: 1,
                    type: "cell",
                    color: "red",
                },
            },
            second: {
                size: 60,
                el: {
                    type: "container",
                    orientation: "vertical",
                    id: 2,
                    first: {
                        size: 50,
                        el: {
                            id: 3,
                            type: "cell",
                            color: "blue",
                        },
                    },
                    second: {
                        size: 50,
                        el: {
                            id: 4,

                            type: "cell",
                            color: "green",
                        },
                    },
                },
            },
        },
        nextId: 5,
    }),
    methods: {
        add() {
            const cell = {
                id: this.nextId++,
                type: "cell",
                color: "pink",
            }

            if (!this.gridData) {
                this.gridData = cell
                return
            }

            this.moveToEdge("horizontal", cell, false)

            this.fixGridData()
        },
        remove(id) {
            if (this.gridData.type === "cell") {
                this.gridData = null
                return
            }

            const cellData = this.findCellAndParentById(id)

            if (cellData) {
                let [, cellParent] = cellData
                cellParent.el = null
            } else if (this.gridData.first.el.id === id) {
                this.gridData = {
                    type: "container",
                    first: {
                        size: 1,
                        el: {
                            ...this.gridData.second.el,
                        },
                    },
                    second: {
                        size: 1,
                        el: null,
                    },
                }
            } else {
                this.gridData = {
                    type: "container",
                    first: {
                        size: 1,
                        el: null,
                    },
                    second: {
                        size: 1,
                        el: {
                            ...this.gridData.first.el,
                        },
                    },
                }
            }

            console.log(JSON.parse(JSON.stringify(this.gridData)))
            this.fixGridData()
            console.log(JSON.parse(JSON.stringify(this.gridData)))
        },
        drop(e, zone) {
            const [cell, cellParent] = this.findCellAndParentById(e.data)

            cellParent.el = null

            switch (zone) {
                case "top":
                    this.moveToEdge("vertical", cell, true)
                    break
                case "bottom":
                    this.moveToEdge("vertical", cell, false)
                    break

                case "left":
                    this.moveToEdge("horizontal", cell, true)
                    break

                case "right":
                    this.moveToEdge("horizontal", cell, false)
                    break
            }
            this.fixGridData()
        },
        moveToEdge(orientation, dragCell, isFirst) {
            const cell = {
                size: 50,
                el: { ...dragCell },
            }

            const rest = {
                size: 50,
                el: { ...this.gridData },
            }

            this.gridData = {
                id: this.nextId++,
                type: "container",
                orientation,
                first: isFirst ? cell : rest,
                second: !isFirst ? cell : rest,
            }
        },
        findCellAndParentById(id) {
            const traverse = (node) => {
                if (!node) {
                    return null
                }

                if (node.first?.el.id === id) {
                    return [node.first.el, node.first]
                }

                if (node.second?.el.id === id) {
                    return [node.second.el, node.second]
                }

                const searchFirst = traverse(node.first?.el)

                if (searchFirst) {
                    return searchFirst
                }

                const searchSecond = traverse(node.second?.el)

                if (searchSecond) {
                    return searchSecond
                }

                return null
            }

            return traverse(this.gridData)
        },
        move(id, dropId, zone) {
            const [dragCell, dragCellParent] = this.findCellAndParentById(id)
            const [dropCell, dropCellParent] = this.findCellAndParentById(
                dropId
            )

            switch (zone) {
                case "replace":
                    return this.replace(
                        dragCell,
                        dragCellParent,
                        dropCell,
                        dropCellParent
                    )
                case "top":
                    return this.rearrange(
                        "vertical",
                        dragCell,
                        dropCell,
                        dropCellParent,
                        dragCellParent
                    )
                case "bottom":
                    return this.rearrange(
                        "vertical",
                        dropCell,
                        dragCell,
                        dropCellParent,
                        dragCellParent
                    )

                case "left":
                    return this.rearrange(
                        "horizontal",
                        dragCell,
                        dropCell,
                        dropCellParent,
                        dragCellParent
                    )

                case "right":
                    return this.rearrange(
                        "horizontal",
                        dropCell,
                        dragCell,
                        dropCellParent,
                        dragCellParent
                    )
            }
        },

        replace(dragCell, dragCellParent, dropCell, dropCellParent) {
            dragCellParent.el = dropCell
            dropCellParent.el = dragCell
        },

        rearrange(
            orientation,
            firstCell,
            secondCell,
            dropCellParent,
            dragCellParent
        ) {
            dropCellParent.el = {
                id: this.nextId++,
                type: "container",
                orientation: orientation,
                first: {
                    size: 50,
                    el: firstCell,
                },
                second: {
                    size: 50,
                    el: secondCell,
                },
            }

            dragCellParent.el = null
            this.fixGridData()
        },

        fixGridData() {
            const traverse = (node, parent) => {
                if (!node) return
                if (node.type === "cell") return

                if (node.first?.el === null) {
                    if (parent) {
                        parent.el = node.second.el
                    } else {
                        this.gridData = node.second.el
                    }
                }

                if (node.second?.el === null) {
                    if (parent) {
                        parent.el = node.first.el
                    } else {
                        this.gridData = node.first.el
                    }
                }

                if (node.first) {
                    traverse(node.first.el, node.first)
                }

                if (node.second) {
                    traverse(node.second.el, node.second)
                }
            }

            traverse(this.gridData, null)
        },
    },
    mounted() {
        // Bad practice move to Vuex
        this.$root.$on("move", this.move)
        this.$root.$on("remove", this.remove)
    },
    destroyed() {
        // Bad practice move to Vuex
        this.$root.$off("move", this.move)
        this.$root.$off("remove", this.remove)
    },
}
</script>

<style scoped>
.grid {
    width: 100%;
    height: 100%;

    position: relative;
}
.button {
    position: absolute;
    top: 10px;
    left: 10px;
    z-index: 1;
}

.drop-zones > * {
    position: absolute;
    border: 2px dotted firebrick;
}

.top {
    top: 0;
    left: 5%;
    right: 5%;
    height: 5%;
}

.bottom {
    bottom: 0;
    left: 5%;
    right: 5%;
    height: 5%;
}

.left {
    top: 0;
    left: 0;
    width: 5%;
    bottom: 0;
}

.right {
    top: 0;
    right: 0;
    width: 5%;
    bottom: 0;
}
</style>
