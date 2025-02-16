<template>
    <div class="pinta-buses">
        <h2>Pinta tu Bus</h2>
        <div class="svg-selector">
            <label for="svg-select">Selecciona un SVG:</label>
            <select id="svg-select" @change="changeSvg($event)">
                <option value="bus">Bus</option>
                <option value="ambulance">Ambulancia</option>
            </select>
        </div>
        <div class="color-palette">
            <div v-for="color in colors" :key="color" :style="{ backgroundColor: color }" class="color-swatch"
                @click="selectColor(color)"></div>
        </div>
        <div v-html="svgContent" @click="paintSvg"></div>
    </div>
</template>

<script>
import busSvg from '@/assets/bus.svg';
import ambulanceSvg from '@/assets/ambulance.svg';

export default {
    name: 'PintaBuses',
    data() {
        return {
            colors: [
                '#FF0000', '#00FF00', '#0000FF', '#FFFF00', '#FFA500', '#800080',
                '#FFC0CB', '#A52A2A', '#8A2BE2', '#5F9EA0', '#D2691E', '#FF7F50',
                '#6495ED', '#DC143C', '#00FFFF', '#00008B'
            ],
            selectedColor: '#000000',
            svgContent: ''
        };
    },
    created() {
        this.loadSvg(busSvg);
    },
    methods: {
        loadSvg(svgFile) {
            fetch(svgFile)
                .then(response => response.text())
                .then(svg => {
                    this.svgContent = svg;
                });
        },
        changeSvg(event) {
            const selectedSvg = event.target.value;
            if (selectedSvg === 'bus') {
                this.loadSvg(busSvg);
            } else if (selectedSvg === 'ambulance') {
                this.loadSvg(ambulanceSvg);
            }
        },
        selectColor(color) {
            this.selectedColor = color;
        },
        paintSvg(event) {
            const target = event.target;
            if (target.tagName === 'rect' || target.tagName === 'circle' || target.tagName === 'path') {
                target.setAttribute('fill', this.selectedColor);
            }
        }
    }
};
</script>

<style scoped>
.pinta-buses {
    text-align: center;
}

.svg-selector {
    margin-bottom: 20px;
}

.color-palette {
    display: flex;
    justify-content: center;
    margin-bottom: 20px;
}

.color-swatch {
    width: 30px;
    height: 30px;
    margin: 0 5px;
    cursor: pointer;
    border: 1px solid #000;
}

svg {
    width: 400px;
    height: 200px;
    cursor: pointer;
}
</style>