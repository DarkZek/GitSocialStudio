<template>
  <div class="preview">
    <div class="img">
        <canvas width="1280" height="640" ref="canvas" id="canvas"/>
    </div>
    <!-- Ensuring fonts are preloaded -->
    <div style="font-family: Open Sans;"><a style="opacity: 0.001; position: absolute; top: -10px;">t</a></div>
    <div style="font-family: PT Sans Caption;"><a style="opacity: 0.001; position: absolute; top: -10px;">t</a></div>
  </div>
</template>

<script>

var canvasHeight = 640;
var canvasWidth = 1280;
var margins = 100;

export default {
    data: function() {
        return {
            name: "Rustcraft",
            description: "Rustcrat is a Minecraft Client,",
            tags: ["rust", "wgpu-rs", "specs ecs"],
            color: "red"
        };
    },
    mounted: function() {
        this.eventHub.$on('setTheme', data => {
            switch (data.theme) {
                case 1: {
                    this.theme_1(data);
                }
            }
        })
    },
    beforeDestroy() {
        this.eventHub.$off('setTheme');
    },
    methods: {
        theme_1: function(data) {
            const canvas = this.$refs.canvas;
            
            var ctx = canvas.getContext("2d");
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Draw background image
            let backgroundRatio = data.background.width / data.background.height;
            // Move left as the slider is changed
            let xShift = -((100 - data.data.backgroundShift) / 100) * (canvasHeight * backgroundRatio);
            ctx.drawImage(data.background, xShift, 0, canvasHeight * backgroundRatio, canvasHeight);
            
            // Gradient
            let graidentLength = (canvasWidth / 2.0) + 100;
            var grd = ctx.createLinearGradient(0, 0, graidentLength, 0);
            grd.addColorStop(0, "rgba(255,255,255,0)");
            grd.addColorStop(1, "white");
            grd.addColorStop(1, "white");

            // Fill with gradient
            ctx.fillStyle = grd;
            ctx.fillRect(0, 0, graidentLength, canvasHeight); 

            // Fill rest
            ctx.fillRect(graidentLength, 0, canvasWidth - graidentLength, canvasHeight);

            // Technically this should be Nimbus Sans, however that is paid
            // Write name
            ctx.fillStyle = "#32363d";
            ctx.font = data.data.titleSize + "px PT Sans Caption";
            let width = ctx.measureText(data.data.repo.full_name.split("/")[1], 0, 0).width;
            ctx.fillText(data.data.repo.full_name.split("/")[1], canvasWidth - margins - width, canvasHeight / 4);

            // Write name
            ctx.fillStyle = "#32363d";
            ctx.font = data.data.titleSize + "px Open Sans";
            let nameWidth = ctx.measureText(data.data.repo.owner.login + "/", 0, 0).width + width;
            ctx.fillText(data.data.repo.owner.login + "/",  canvasWidth - margins - nameWidth, canvasHeight / 4); 

            // Write description
            ctx.fillStyle = "#7d7d81";
            ctx.font = "35px Open Sans";
            var lineGap = 50;

            let desc = data.data.repo.description;
            for (var i = 0; i < 3; i++) {
                // Find how many words we can fit
                let line = "";
                while (ctx.measureText(line, 0, 0).width < canvasWidth - nameWidth) {
                    let nextSpaceIndex = desc.substr(line.length).indexOf(" ") + line.length + 2;
                    line += desc.substr(line.length, nextSpaceIndex);
                    if (line == desc) {break;}
                }

                desc = desc.substr(line.length);

                ctx.fillText(line, canvasWidth - margins - nameWidth, (canvasHeight / 4) + (lineGap * 2) + (lineGap * i)); 
            }

            // Bottom line
            ctx.fillStyle = data.data.color;
            ctx.fillRect(0, canvasHeight - 30, canvasWidth, 30);
        }
    }
}
</script>

<style scoped>

.preview {

}
.img {
    position: absolute;
    left: 0px;
    right: 0px;
    top: 20%;
    margin: auto;
    background-color: var(--light-grey);
    width: 640px;
    height: 320px;
    border-radius: 10px;
    box-shadow: 0 0 15px grey;
}
canvas {
    border-radius: 10px;
    overflow: hidden;
    width: 100%;
    height: 100%;
}
</style>
