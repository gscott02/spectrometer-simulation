<script>
    // 1. Import Chart.js so you can use the global Chart object
    import Chart from 'chart.js'
    // 2. Import the `generateChart()` method to create the vue component.
    import { generateChart } from 'vue-chartjs'

    // 3. Extend on of the default charts
    // http://www.chartjs.org/docs/latest/developers/charts.html
    //From https://stackoverflow.com/a/45172506/10458926
    Chart.defaults.LineWithLine = Chart.defaults.line;
    Chart.controllers.LineWithLine = Chart.controllers.line.extend({
        draw: function(ease) {
            Chart.controllers.line.prototype.draw.call(this, ease);

            if (this.chart.tooltip._active && this.chart.tooltip._active.length) {
                let activePoint = this.chart.tooltip._active[0],
                    ctx = this.chart.ctx,
                    x = activePoint.tooltipPosition().x,
                    topY = this.chart.scales['y-axis-0'].top,
                    bottomY = this.chart.scales['y-axis-0'].bottom;

                // draw line
                ctx.save();
                ctx.beginPath();
                ctx.moveTo(x, topY);
                ctx.lineTo(x, bottomY);
                ctx.lineWidth = 2;
                ctx.strokeStyle = '#07C';
                ctx.stroke();
                ctx.restore();
            }
        }
    });

    //Adapted from: https://stackoverflow.com/questions/38493564/chart-area-background-color-chartjs
    Chart.pluginService.register({
        beforeDraw: function (chart, easing) {
            let helpers = Chart.helpers;
            let ctx = chart.chart.ctx;
            let chartArea = chart.chartArea;

            let grd = ctx.createLinearGradient(0, 0, chartArea.right*400/520, 0);  // 400/520 represents the fact that the visible spectrum is ~380-780 nm, but we are plotting 380-900 nm.
            for(let i=380; i<=780; i++) {
                grd.addColorStop((i-380)/400, wavelengthToColor(i)[0]);
            }
            //grd.addColorStop(0, "black");
            grd.addColorStop(400/520, "white");
            grd.addColorStop(1, "white");

            ctx.save();
            ctx.fillStyle = grd;//chart.config.options.chartArea.backgroundColor;
            ctx.fillRect(chartArea.left, chartArea.top, chartArea.right - chartArea.left, chartArea.bottom - chartArea.top);
            ctx.restore();
        }
    });

    // takes wavelength in nm and returns an rgba value
    // https://scienceprimer.com/javascript-code-convert-light-wavelength-color
    function wavelengthToColor(wavelength) {
        var r,
            g,
            b,
            alpha,
            colorSpace,
            wl = wavelength,
            gamma = 1;

        let R, G, B;

        if (wl >= 380 && wl < 440) {
            R = -1 * (wl - 440) / (440 - 380);
            G = 0;
            B = 1;
        } else if (wl >= 440 && wl < 490) {
            R = 0;
            G = (wl - 440) / (490 - 440);
            B = 1;
        } else if (wl >= 490 && wl < 510) {
            R = 0;
            G = 1;
            B = -1 * (wl - 510) / (510 - 490);
        } else if (wl >= 510 && wl < 580) {
            R = (wl - 510) / (580 - 510);
            G = 1;
            B = 0;
        } else if (wl >= 580 && wl < 645) {
            R = 1;
            G = -1 * (wl - 645) / (645 - 580);
            B = 0.0;
        } else if (wl >= 645 && wl <= 780) {
            R = 1;
            G = 0;
            B = 0;
        } else {
            R = 0;
            G = 0;
            B = 0;
        }

        // intensty is lower at the edges of the visible spectrum.
        if (wl > 780 || wl < 380) {
            alpha = 0;
        } else if (wl > 700) {
            alpha = (780 - wl) / (780 - 700);
        } else if (wl < 420) {
            alpha = (wl - 380) / (420 - 380);
        } else {
            alpha = 1;
        }

        colorSpace = ["rgba(" + (R * 100) + "%," + (G * 100) + "%," + (B * 100) + "%, " + alpha + ")", R, G, B, alpha]

        // colorSpace is an array with 5 elements.
        // The first element is the complete code as a string.
        // Use colorSpace[0] as is to display the desired color.
        // use the last four elements alone or together to access each of the individual r, g, b and a channels.

        return colorSpace;

    }

    // 4. Generate the vue-chartjs component
    // First argument is the chart-id, second the chart type.
    const CustomLine = generateChart('custom-line', 'LineWithLine');

    // 5. Extend the CustomLine Component just like you do with the default vue-chartjs charts.

    export default {
        extends: CustomLine,

        props: ['chartData','options'],

        mounted () {
            //renderChart function renders the chart with the datacollection and options object.
            this.renderChart(this.chartData, this.options)
        },
        watch: {
            chartData: {
                //deep: true,
                handler() {
                    this.renderChart(this.chartData, this.options)
                    //this.$data._chart.update(this.chartData, this.options)
                    //The update does not work correctly as suggested by the documentation, so calling render instead
                    //https://github.com/apertureless/vue-chartjs/issues/582
                }
            },
            options () {
                this.renderChart(this.chartData, this.options);
            },
        }
    }
</script>