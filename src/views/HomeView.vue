<script setup>
import { VueFlow, useVueFlow } from "@vue-flow/core";
import { Background } from "@vue-flow/background";
import { Controls } from "@vue-flow/controls";
import { MiniMap } from "@vue-flow/minimap";
import { ref } from "vue";
import { MarkerType, Position } from "@vue-flow/core";
import axios from "axios";
import { onMounted } from "vue";
const { onConnect, addEdges } = useVueFlow();
// const elements = ref(initialElements);
const apiResponse1 = {
    current: {
        jobName: "batch01",
        status: "11",
    },
    trailings: [
        {
            jobName: "batch02",
            status: "11",
        },
        {
            jobName: "batch02-1",
            status: "11",
        },
    ],
};

const apiResponse2 = {
    current: {
        jobName: "batch02",
        status: "11",
    },
    precedings: {
        jobName: "batch01",
        status: "11",
    },
    trailings: [
        {
            jobName: "batch03",
            status: "11",
        },
        {
            jobName: "batch03-1",
            status: "11",
        },
    ],
};

const apiResponse3 = {
    current: {
        jobName: "batch02-1",
        status: "11",
    },
    precedings: {
        jobName: "batch01",
        status: "11",
    },
    trailings: [
        {
            jobName: "batch04",
            status: "11",
        },
        {
            jobName: "batch04-1",
            status: "11",
        },
    ],
};

const tobeRenderedElements = ref([]);
let lastPosition = { x: 0, y: 50 };
let lastPreceding = "";
onMounted(async () => {
    const queueForBFS = [];
    tobeRenderedElements.value.push({
        id: apiResponse1.current.jobName,
        type: "output",
        label: apiResponse1.current.jobName,
        position: lastPosition,
        sourcePosition: Position.Right,
    });
    apiResponse1.trailings.forEach((each) => {
        queueForBFS.push(each);
    });
    lastPreceding = apiResponse1.current.jobName;
    lastPosition = { ...lastPosition, x: lastPosition.x + 100 };

    while (queueForBFS.length > 0) {
        console.log("###### queueForBFS: ", queueForBFS);
        const head = queueForBFS.shift();
        console.log("###### head: ", head);
        const trailingBatchInfo = (await axios.get(`/test/${head.jobName}`))
            .data;
        console.log("###### tailingBatchInfo: ", trailingBatchInfo);
        if (!trailingBatchInfo.precedings.includes(lastPreceding)) {
            lastPosition = { ...lastPosition, x: lastPosition.x + 100 };
        }
        tobeRenderedElements.value.push({
            id: head.jobName,
            type: "default",
            label: head.jobName,
            position: { ...lastPosition, y: lastPosition.y + 50 },
            sourcePosition: Position.Left,
            targetPosition: Position.Right,
        });
    }
    console.log("####### tobeRenderedElements: ", tobeRenderedElements.value);
});

onConnect((params) => addEdges(params));
</script>

<template>
    <VueFlow
        v-model="tobeRenderedElements"
        class="basicflow"
        :default-edge-options="{ type: 'smoothstep' }"
        :default-viewport="{ zoom: 1.5 }"
        :min-zoom="0.2"
        :max-zoom="4"
        fit-view-on-init
    >
        <Background pattern-color="#aaa" gap="8" />

        <MiniMap />

        <Controls />
    </VueFlow>
</template>

<style>
@import "https://cdn.jsdelivr.net/npm/@vue-flow/core@1.27.1/dist/style.css";
@import "https://cdn.jsdelivr.net/npm/@vue-flow/core@1.27.1/dist/theme-default.css";
@import "https://cdn.jsdelivr.net/npm/@vue-flow/controls@latest/dist/style.css";
@import "https://cdn.jsdelivr.net/npm/@vue-flow/minimap@latest/dist/style.css";
@import "https://cdn.jsdelivr.net/npm/@vue-flow/node-resizer@latest/dist/style.css";

html,
body,
#app {
    margin: 0;
    height: 100%;
}

#app {
    text-transform: uppercase;
    font-family: "JetBrains Mono", monospace;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
}

.vue-flow__minimap {
    transform: scale(75%);
    transform-origin: bottom right;
}
</style>
