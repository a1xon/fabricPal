<template>
    <div class="my-5"></div>
    <v-row>
        <v-col cols="12" class="dynamicsegment d-flex justify-center align-center pattern">
            <div class="dropbox" v-if="processingStatus === 'userinput'">
                <input
                    class="input-file px-2 py-1 text-sm font-semibold rounded hover:(text-white bg-sky-100 border-transparent) focus:(outline-none ring-2 ring-sky-200)"
                    type="file"
                    @change="patternFileInput($event)"
                    accept="image/png, image/jpeg"
                />
                <p class="m-5">
                    Drag your sewing pattern here to begin
                    <br />or click to browse
                </p>
            </div>
            <h2 v-if="!['userinput', ''].includes(processingStatus)">{{ processingStatus }}</h2>
            <svg
                id="svg"
                version="2"
                :width="svg.width"
                :height="svg.height"
                :viewbox="`0 0 ${svg.width} ${svg.height}`"
                xmlns="http://www.w3.org/2000/svg"
                @click="handleSVGclick"
            >
                <defs>
                    <!-- svg pattern not sewing pattern - dont get confused here -->
                    <pattern
                        v-for="fabric of fabrics"
                        :id="fabric.id"
                        patternUnits="userSpaceOnUse"
                        :width="fabric.adjustedWidth"
                        :height="fabric.adjustedHeight"
                    >
                        <image
                            :href="fabric.image.src"
                            :x="0"
                            :y="0"
                            :width="fabric.adjustedWidth"
                            :height="fabric.adjustedHeight"
                        />
                    </pattern>
                </defs>
            </svg>
        </v-col>
    </v-row>
</template>

<style>
.pattern {
    height: 90vh;
}
.pattern > svg {
    max-width: 100%;
    max-height: 100%;
}
</style>

<script>
import { ref, reactive, computed, inject } from 'vue';
import { Potrace } from '../plugins/potrace'
import { SVGPathElementExtender } from '../plugins/path-data-polyfill'

class Pattern {
    constructor(image) {
        this.id = Math.random().toString(16).slice(2)
        this.image = image;
        this.selected = false;
    }
}

const randomColor = () => '#' + ('00000' + (Math.random() * (1 << 24) | 0).toString(16)).slice(-6)

export default {
    setup() {
        let processingStatus = ref("userinput");
        let svg = reactive({ width: 0, height: 0, paths: [] });
        //extend SVGPathElement
        SVGPathElementExtender();
        const fabrics = inject('getFabrics');
        const selectedFabric = inject('selectedFabric');

        const handleSVGclick = (e) => {
            if (e.target.matches('path') && selectedFabric?.value?.id) {
                e.target.setAttribute('fill', `url(#${selectedFabric.value.id})`);
            }
        };
        const patternFileInput = async ($event) => {
            for (const file of $event?.target?.files) {
                processingStatus.value = "vectorizing";
                Potrace.loadImageFromFile(file);
                Potrace.setParameter({
                    turnpolicy: 'black',
                    turdsize: 25,
                    optcurve: false,
                    alphamax: 0,
                    opttolerance: 0,
                });
                Potrace.process(() => {
                    breakUpFile();
                })
            }
        }
        const breakUpFile = () => {
            processingStatus.value = "breaking up";
            const svgElement = document.getElementById('svg');
            const { svgPath, width, height } = Potrace.getSVG(1);
            svg.width = width;
            svg.height = height;

            /// create svg object to take apart
            const pathContainer = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
            pathContainer.innerHTML = svgPath;
            const tracedSvgPath = pathContainer.querySelector('path');

            // normalize should be used to get back absolute segments
            const pathsData = tracedSvgPath.getPathData({ normalize: true }).reduce((acc, seg) => {
                let pathData = seg.type === 'M' ? [] : acc.pop()
                seg.values = seg.values.map(v => Math.round(v * 1000) / 1000)
                pathData.push(seg)
                acc.push(pathData)
                return acc
            }, []);

            processingStatus.value = "";
            pathsData.sort((a, b) => b.length - a.length);
            
            for (const [index, d] of pathsData.entries()) {
                const path = document.createElementNS('http://www.w3.org/2000/svg', 'path')
                path.setPathData(d);
                let fill = "#EEEEEE";
                if (index === 0) {
                    fill = `#BBBBBB`;
                } else if (selectedFabric?.value?.id) {
                    fill = `url(#${selectedFabric.value.id})`;
                }
                path.setAttribute('fill', fill);


                svg.paths.push(path);
                svgElement.appendChild(path);
                //console.log(path);
            }
        }
        return { svg, patternFileInput, processingStatus, fabrics, handleSVGclick }
    }
}
</script>