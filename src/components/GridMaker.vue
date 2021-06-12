<template lang="pug">
    .grid-maker
        .actions
            .file
                label
                    span.button Select Image
                    input(type="file" style="display: none;" @change="image_changed")
                    br.clearfix
            .grids
                label X
                    input(type="number" min="1" max="10" step="1" v-model.number="panes_x")
                label Y
                    input(type="number" min="1" max="10" step="1" v-model.number="panes_y")
            .colors
                select(v-model="grid_color")
                    option(value="black") black
                    option(value="grey") grey
                    option(value="red") red
                    option(value="green") green
                    option(value="blue") blue
                    option(value="white") white

        .info
            span(v-text="`width: ${image_info.width}, height: ${image_info.height}`")

        svg#svg(:width="image_info.width" :height="image_info.height" :viewBox="viewBox")
            image(:href="image")
            g.grid
                line(v-for="l in lines" :x1="l.x1" :y1="l.y1" :x2="l.x2" :y2="l.y2" :stroke="grid_color")
        br
        a(v-if="image" href="#" @click="download") create download link
        br
        a#download(v-show="image && show_dl_link" target="_blank") Download
        canvas#canvas.invisible
</template>

<script lang="ts">
import {Component, Prop, Vue} from 'vue-property-decorator';

declare interface FileSelectEvent {
    target: { files: FileList }
}

@Component
export default class GridMaker extends Vue {
    image_source: string = '';

    get image(): string {
        return this.image_source;
    }

    set image(value: string) {
        this.image_source = value;
    }

    image_info = {
        width: 0,
        height: 0
    }

    panes_x: number = 3;
    panes_y: number = 3;

    grid_color = 'black';

    show_dl_link: boolean = false;

    get lines(): any[] {
        const lines_v: any[] = [];
        const lines_h: any[] = [];
        const span_x: number = this.image_info.width / this.panes_x;
        const span_y: number = this.image_info.height / this.panes_y;

        // vertical
        for (let i: number = 1; i < this.panes_x; i++) {
            const x = i * span_x;
            lines_v.push({
                x1: x,
                y1: 0,
                x2: x,
                y2: this.image_info.height
            });
        }

        // horizontal
        for (let i: number = 1; i < this.panes_y; i++) {
            const y = i * span_y;
            lines_v.push({
                x1: 0,
                y1: y,
                x2: this.image_info.width,
                y2: y
            });
        }
        return [...lines_v, ...lines_h];
    }

    get viewBox(): string {
        return `0 0 ${this.image_info.width} ${this.image_info.height}`;
    }

    image_changed(e: FileSelectEvent) {
        const file = e.target.files[0];

        if (!file.type.startsWith('image/')) {
            return;
        }
        console.log(file);
        const img = document.createElement("img");
        img.classList.add("obj");

        const reader = new FileReader();
        reader.onload = ((aImg) => {
            return (e: any) => {
                aImg.src = e.target.result;
                this.image = e.target.result;

                Vue.nextTick(() => {
                    this.image_info.width = aImg.naturalWidth;
                    this.image_info.height = aImg.naturalHeight;
                });
            };
        })(img);
        reader.readAsDataURL(file);
    }

    download(): void {
        const $svg = document.querySelector('#svg')!;
        const $img = new Image();
        const xml = new XMLSerializer().serializeToString($svg);
        const b64Start = "data:image/svg+xml;base64,";
        const image64 = b64Start + btoa(xml);

        $img.onload = () => {
            const sx = 0;
            const sy = 0;
            const sw = this.image_info.width;
            const sh = this.image_info.height;
            const dx = 0;
            const dy = 0;
            const dw = sw;
            const dh = sh;

            const canvas: HTMLCanvasElement = document.getElementById("canvas")! as HTMLCanvasElement;
            canvas.width = dw;
            canvas.height = dh;
            const ctx = canvas.getContext!("2d")!;

            ctx.fillStyle = 'black';
            ctx.fillRect(0, 0, dw, dh);
            ctx.drawImage($img, sx, sy, sw, sh, dx, dy, dw, dh);
            const source: string = canvas.toDataURL('image/png');
            const $dlLink = document.getElementById('download')!;
            $dlLink.setAttribute('href', source);
            $dlLink.setAttribute('download', 'grid1.png');
            this.show_dl_link = true;
        }

        $img.src = image64;
    }
}
</script>

<style scoped lang="less">
.actions {
    margin-bottom: 10px;
}

img, svg {
    border: 1px solid grey;
}

.button {
    border: 1px solid green;
    background-color: lightgreen;
    cursor: pointer;
    border-radius: 5px;
    padding: 5px;

    &:active {
        position: relative;
        top: 1px;
    }
}

g.grid {
    line {
        stroke-width: 1px;
    }
}

.file, .grids {
    display: block;
    margin-bottom: 10px;

}

.invisible {
    display: none;
}
</style>
