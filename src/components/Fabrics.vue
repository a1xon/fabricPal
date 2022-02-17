<template>
    <v-row>
        <v-col cols="12" class="d-flex justify-center">
            <img src="../assets/img/logo.svg" class="mainlogo dynamicsegment px-5 py-5" />
        </v-col>
    </v-row>
    <v-row v-if="selectedFabric">
        <v-col cols="12" class="dynamicsegment">
            <h2>Current</h2>
            <div class="d-flex justify-center align-center">
                <div class="text-center">
                    <v-btn
                        width="128"
                        height="128"
                        class="selectedfabric--img"
                        elevation="6"
                        icon
                        :style="`background-image: url(${selectedFabric.image.src})`"
                    ></v-btn>
                    <div class="my-5"></div>
                    <input
                        type="range"
                        id="size"
                        v-model="selectedFabric.size"
                        name="size"
                        min="0"
                        max="100"
                        class="mx-5"
                        style="accent-color: #02182b;"
                    />
                    <label for="size">Size</label>
                </div>
            </div>
        </v-col>
    </v-row>
    <div class="my-5"></div>
    <v-row v-if="getFabrics.length >= 1">
        <v-col cols="12" class="dynamicsegment">
            <h2>Fabrics</h2>
            <div class="d-flex flex-wrap justify-space-around">
                <v-btn
                    v-for="fabric of getFabrics"
                    icon
                    size="x-large"
                    elevation="6"
                    class="fabric--btn mx-2 my-2"
                    :style="`background-image: url(${fabric.image.src})`"
                    @click="selectFabric(fabric.id)"
                ></v-btn>
            </div>
        </v-col>
    </v-row>
    <div class="my-5"></div>
    <v-row>
        <v-col cols="12" class="dynamicsegment">
            <h2>Upload Fabrics</h2>
            <div class="dropbox">
                <input
                    class="input-file px-2 py-1 text-sm font-semibold rounded hover:(text-white bg-sky-100 border-transparent) focus:(outline-none ring-2 ring-sky-200)"
                    type="file"
                    @change="fabricFileInput($event)"
                    multiple
                    accept="image/png, image/jpeg"
                />
                <p class="m-5">
                    Drag your fabric(s) here to begin
                    <br />or click to browse
                </p>
            </div>
        </v-col>
    </v-row>
</template>

<style scoped>
.selectedfabric--img {
    background-position: center !important;
    background-size: cover !important;
}

.fabric--btn {
    background-position: center !important;
    background-size: cover !important;
}

.mainlogo {
    max-width: 300px;
    min-width: 200px;
    width: 20vw;
}
</style>

<script>
import { ref, computed, inject } from 'vue';
import blank from "../assets/img/fabrics/blank.png"
import fabric0 from "../assets/img/fabrics/fabric0.jpg"
import fabric1 from "../assets/img/fabrics/fabric1.jpg"
import fabric2 from "../assets/img/fabrics/fabric2.jpg"
import fabric3 from "../assets/img/fabrics/fabric3.png"

class Fabric {
    constructor(image, width, height) {
        this.id = Math.random().toString(16).slice(2)
        this.image = image;
        this.selected = false;
        this.width = width || image.width;
        this.height = height || image.height;
        this.size = ref(50);
        this.adjustedWidth = computed(() => Math.max(Math.floor(this.width * (this.size.value / 100)),0.01));
        this.adjustedHeight = computed(() => Math.max(Math.floor(this.height * (this.size.value / 100)),0.01));
    }
}
export default {
    name: 'Patterns',
    setup() {

        const store =  inject('useProvideStore');
        console.log(store());
        const {getFabrics, pushFabrics, selectFabric, selectedFabric} = store();

        pushFabrics([
            new Fabric({ src: blank, width: 200, height: 200 }),
            new Fabric({ src: fabric0, width: 800, height: 600 }),
            new Fabric({ src: fabric1, width: 600, height: 800 }),
            new Fabric({ src: fabric2, width: 605, height: 605 }),
            new Fabric({ src: fabric3, width: 400, height: 300 })
        ]);

        const fabricFileInput = async ($event) => {
            for (const file of $event?.target?.files) {
                const image = new Image();
                image.src = URL.createObjectURL(file);
                image.onload = () => {
                    pushFabrics([new Fabric(image)]);
                }
            };
        }

        return { getFabrics, fabricFileInput, selectFabric, selectedFabric };
    }
}
</script>