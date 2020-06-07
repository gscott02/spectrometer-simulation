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