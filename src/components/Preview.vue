<template>
  <div class="preview">
    <div class="img">
        <canvas width="1280" height="640" ref="canvas" id="canvas"/>
    </div>
    <!-- Ensuring fonts are preloaded -->
    <div style="font-family: Open Sans;"><a style="opacity: 0.001; position: absolute; top: -10px;">t</a></div>
    <div style="font-family: PT Sans Caption;"><a style="opacity: 0.001; position: absolute; top: -10px;">t</a></div>
    <img ref="githubLogo" src="@/assets/Github.svg" style="opacity: 0.001; position: absolute; top: -100px;">
  </div>
</template>

<script>

var canvasHeight = 640;
var canvasWidth = 1280;
var margins = 100;

export default {
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

            // Draw default white
            ctx.fillStyle = "white";
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Draw background image
            let backgroundRatio = data.backgroundImg.width / data.backgroundImg.height;
            // Move left as the slider is changed
            let xShift = -((100 - data.data.backgroundShift) / 100) * (canvasHeight * backgroundRatio);
            ctx.drawImage(data.backgroundImg, xShift, 0, canvasHeight * backgroundRatio, canvasHeight);
            
            // Gradient
            let graidentLength = (canvasWidth / 2.0) + 100;
            // Make gradient smoother
            var grd = ctx.createLinearGradient(0, 0, graidentLength, 0);
            grd.addColorStop(0, "rgba(255,255,255,0.2)");
            grd.addColorStop(0.1, "rgba(255,255,255,0.25)");
            grd.addColorStop(0.25, "rgba(255,255,255,0.3)");
            grd.addColorStop(0.35, "rgba(255,255,255,0.4)");
            grd.addColorStop(0.5, "rgba(255,255,255,0.6)");
            grd.addColorStop(0.6, "rgba(255,255,255,0.75)");
            grd.addColorStop(0.7, "rgba(255,255,255,0.9)");
            grd.addColorStop(0.8, "rgba(255,255,255,0.98)");
            grd.addColorStop(1, "rgba(255,255,255,1)");
            ctx.fillStyle = grd;
            ctx.fillRect(0, 0, graidentLength, canvasHeight); 

            // Fill rest
            ctx.fillStyle = "white";
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
            var lines = 0;

            let descriptionX = canvasWidth - margins - nameWidth - data.data.descriptionShift;

            let desc = data.data.repo.description;
            for (var i = 0; i < 3; i++) {
                // Find how many words we can fit
                let line = "";
                
                while (ctx.measureText(line, 0, 0).width < nameWidth + parseFloat(data.data.descriptionShift) - margins || line == desc) {
                    let nextSpaceIndex = desc.substr(line.length + 1).indexOf(" ") + line.length + 1;
                    if (desc.startsWith(" ")) { desc = desc.substr(1); }
                    line += desc.substr(line.length, nextSpaceIndex - line.length);
                }

                desc = desc.substr(line.length);
                if (i == 2) { line += "..."; }
                ctx.fillText(line, descriptionX, (canvasHeight / 4) + (lineGap * 2) + (lineGap * i)); 
                lines += 1;
            }

            // Bottom line
            ctx.fillStyle = data.data.color;
            ctx.fillRect(0, canvasHeight - 30, canvasWidth, 30);

            // Github logo
            ctx.drawImage(this.$refs.githubLogo, canvasWidth - margins - 40, canvasHeight - margins - 40, 70, 70);

            // Draw project logo
            let logoRatio = data.logoImg.width / data.logoImg.height;
            var scale = (data.data.logoSize / 100);
            var startingSize = canvasHeight / 2.5;
            // Scale
            startingSize *= scale;
            
            let roundImg = function roundedImage(ctx, x, y, width, height, radius) {
                ctx.beginPath();
                ctx.moveTo(x + radius, y);
                ctx.lineTo(x + width - radius, y);
                ctx.quadraticCurveTo(x + width, y, x + width, y + radius);
                ctx.lineTo(x + width, y + height - radius);
                ctx.quadraticCurveTo(x + width, y + height, x + width - radius, y + height);
                ctx.lineTo(x + radius, y + height);
                ctx.quadraticCurveTo(x, y + height, x, y + height - radius);
                ctx.lineTo(x, y + radius);
                ctx.quadraticCurveTo(x, y, x + radius, y);
                ctx.closePath();
            };
            ctx.save();
            roundImg(ctx, margins + 20 - (data.data.logoSize / 2.0) + parseFloat(data.data.logoShift), (canvasHeight / 4) - data.data.titleSize + 20, startingSize, startingSize / logoRatio, 15);
            ctx.clip();
            ctx.drawImage(data.logoImg, margins + 20 - (data.data.logoSize / 2.0) + parseFloat(data.data.logoShift), (canvasHeight / 4) - data.data.titleSize + 20, startingSize, startingSize / logoRatio);
            ctx.restore();
            
            var drawRoundedBackground = function(x, y, width, height) {
                // Account for sides of circle
                width -= height;

                // Center
                ctx.fillStyle = data.data.color;
                ctx.fillRect(x, y, width, height);

                ctx.beginPath();
                // Left circle
                ctx.arc(x, y + (height / 2), height / 2, 0, 2 * Math.PI);
                ctx.fill(); 

                // Right circle
                ctx.arc(x + width, y + (height / 2), height / 2, 0, 2 * Math.PI);
                ctx.fill(); 
            }

            // Draw tags
            var workingX = 0;
            for (i = 0; i < Math.min(4, data.data.repo.topics.length); i++) {
                let topic = data.data.repo.topics[i];

                let width = ctx.measureText(topic, 0, 0).width;

                drawRoundedBackground(descriptionX + workingX + 30, (canvasHeight / 4) + (lineGap * 2) + (lineGap * (lines - 0.35)), width + 65, 65);
                ctx.fillStyle = "white";
                ctx.fillText(topic, descriptionX + workingX + 30, (canvasHeight / 4) + (lineGap * 2) + (lineGap * (lines + 0.5)));
                workingX += width + 90;
            }

            ctx.save();
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
