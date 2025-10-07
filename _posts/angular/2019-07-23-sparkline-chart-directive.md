---
title: "Creating Sparkline Chart Directive with ng2-nvd3"
date: 2019-07-23 16:11:00 +0900
comments: true
categories: angular
tags: [sparkline, chart, directive, nvd3, ng2-nvd3]
---

I'm attempting to create a Sparkline Chart because most typical chart libraries don't support it, especially those compatible with Angular. I plan to build this chart myself and leverage `ng2-nvd3` since I prefer charts rendered with this library. To ensure ease of reuse, I'll implement it as an Angular Directive.

## Prerequisites

To use nvd3 with Angular, you must install ng2-nvd3:

```bash
npm install ng2-nvd3
```

## Key Features

- Sparkline Charts differentiate themselves by using a single line where areas above and below a threshold have different colors. Standard line charts lack this feature.
- The implementation strategy involves overlaying two charts. The fill area of one chart is inverted, and both are clipped based on the threshold value.
- Rather than relying on nvd3 commands, this directive directly renders SVG elements to the DOM.

## Directive Parameters

These options are available when invoking the directive. Parameters without defaults are mandatory.

```html
{% raw %}
<div id="sparklineThreshold"
        [sparklineThreshold]="sparklineThresholddata"  
        [height]="100" 
        [width]="400" 
        [type]="linear" 
        [threshold]="100" 
        [strokecolors]="['red','blue']" 
        [fillcolors]="['#da343452', '#c7daea']">
</div>
{% endraw %}
```

### Parameter Details

- `id`:  An ID is required for the DOM element to allow multiple directives to be used simultaneously on a single page.
- `sparklineThreshold`: The directive name. Pass chart data as an `Array<number>`.
- `height`: The height of the SVG element.
- `width`: The width of the SVG element, which determines the x-axis division based on the data length.
- `type`: Either `linear` (default) for straight lines or `basis` for curved lines.
- `threshold`: The reference point. The colors of the chart are determined based on this value.
- `strokecolors`: An array containing the line colors for above and below the threshold. Defaults to `['red','blue']`.
- `fillcolors`: An array containing the fill colors for above and below the threshold. Defaults to `['#da343452', '#c7daea']`.

### Data Structure

- `threshold`: This value is constrained within the min and max values of the dataset, and then calculated as a percentage.

```tsx
this.maxData = Math.max(...this.rawData);
this.minData = Math.min(...this.rawData);

// Ensure the threshold does not exceed max/min bounds
let threshold = (this.rawThreshold > this.maxData) ? this.maxData : this.rawThreshold;
threshold = (this.rawThreshold < this.minData) ? this.minData : this.rawThreshold;
this.threshold = Math.round((1 - (this.maxData - threshold) / this.maxData) * 100);
```

- x-axis: Based on the specified width.
- y-axis: Calculated as a percentage based on the maximum data value.

```tsx
const dataLength = this.rawData.length;
const step = Math.round(this.width / this.rawData.length);
let i = 0;

for (const item of this.rawData) {
    this.data.push({
        x: i, y: Math.round((1 - (this.maxData - item) / this.maxData) * 100)
    });
    i += step;
}
```

### Chart Rendering

- The process involves drawing two identical charts.
- The domain is set from the minimum to the maximum data value.
- The range spans from the specified height down to 0.

**Upper Chart**

- The `clipPath` has a y-coordinate of 0 and a height corresponding to the scaled threshold.
- This `clipPath` is applied to the path.
- Only the data above the threshold is visible.

**Lower Chart**

- The `clipPath`'s y-coordinate is set to the scaled threshold, with a height up to the maximum data value.
- This `clipPath` is applied to the path.
- Only the data below the threshold is visible.

## Complete Source Code

```tsx
import { Directive, Input, ViewContainerRef, AfterViewInit } from '@angular/core';
declare let d3: any;

@Directive({
    selector: '[sparklineThreshold]'
})

export class SparkLineThresholdDirective implements AfterViewInit {
    rawData = [];
    data = [];
    maxData = 0;
    minData = 0;
    maxWidth = 0;
    rawThreshold = 0;
    threshold = 0;
    topStrokeColor = 'red';
    bottomStrokeColor = 'blue';
    topFillColor = '#da343452';
    bottomFillColor = '#c7daea';
    elementId;

    // Actual data = Array of Numbers
    @Input('sparklineThreshold') set setData(data: Array<number>) {
      this.rawData = data;
      
      // Test Data
     // const numPoints = 50;
     //   for (let i = 1; i < numPoints; i++) {
     //       const rnd = Math.floor(Math.random() * (250)) + 1;
     //      this.rawData.push(rnd);
      //  }
    }

    @Input('width') width: number;
    @Input('height') height: number;
    @Input('type') type: string;
    @Input('threshold') set setThreshold(threshold: number) { this.rawThreshold = threshold; }
    @Input('strokecolors') set setStrokeTopColor(colors: Array<string>) {
        this.topStrokeColor = colors[0];
        this.bottomStrokeColor = colors[1];
    }
    @Input('fillcolors') set setFillColors(colors: Array<string>) {
        this.topFillColor = colors[0];
        this.bottomFillColor = colors[1];
    }

    constructor(private viewContainer: ViewContainerRef) {
        this.elementId = this.viewContainer.element.nativeElement.id;
    }

    // Rawdata and ID are required
    ngAfterViewInit() {
        if (!this.rawData || !this.rawData.length) {
            console.log('rawData are required');
            return;
        }
        if (!this.elementId) {
            console.error('element Id is required');
            return;
        }

        // Initial values
        this.maxData = Math.max(...this.rawData);
        this.minData = Math.min(...this.rawData);
        // threshold does not exceed the max range to chang
```

```tsx
e the percentage
        let threshold = (this.rawThreshold > this.maxData) ? this.maxData : this.rawThreshold;
        threshold = (this.rawThreshold < this.minData) ? this.minData : this.rawThreshold;
        this.threshold = (100 - Math.round(((this.maxData - threshold) / this.maxData) * 100));

        this.type = this.type ? this.type : 'linear';
        this.setSparklineData();
        this.drawSparkLine();

        // resize https://github.com/novus/nvd3/issues/645
    }

    // Mapping rawData
    setSparklineData() {
        const dataLength = this.rawData.length;
        const step = Math.round(this.width / this.rawData.length);

        let i = 0;
        for (const item of this.rawData) {
            this.maxWidth = step * i;

            this.data.push({
                x: this.maxWidth, y: (100 - Math.round(((this.maxData - item) / this.maxData) * 100))
            });

            i++;
        }
    }

    // Drawing
    drawSparkLine() {
        const svg = d3.select(this.viewContainer.element.nativeElement).append('svg');
        svg.attr('width', this.width).attr('height', this.height + 5); 
        // Add a margin of about 5 because there is an invisible margin.
        // Otherwise, the end point may be hidden.

        const draw = svg.append('g');

        const yScale = d3.scale.linear()
        .domain([this.minData, this.maxData])
        .range([this.height, 0]);

        // top
        const setTopLine = d3.svg.area()
        .x((d) => d.x)
        .y((d) => yScale(d.y))
        .y0(this.height)
        .interpolate(this.type);

        draw.append('clipPath')
        .attr('id', this.elementId + '-top')
        .append('rect')
        .attr('x', 1) // Give 1 to hide the left starting point because it starts at 0 point.
        .attr('y', 0)
        .attr('rx', 0)
        .attr('ry', 0)
        .attr('height', yScale(this.threshold))
        .attr('width', this.maxWidth - 2); // Subtract 2 to hide the part that converges to 0 at the end po
```

```tsx
int.

        draw.append('path')
        .data([this.data])
        .attr('d', setTopLine)
        .attr('clip-path', 'url(#' + this.elementId + '-top)')
        .attr('stroke', this.topStrokeColor)
        .attr('stroke-width', 1)
        .attr('fill', this.topFillColor);

        // bottom
        const setBottomLine = d3.svg.area()
        .x((d) => d.x)
        .y((d) => yScale(d.y))
        .y0(this.threshold)
        .interpolate(this.type);

        draw.append('clipPath')
        .attr('id', this.elementId + '-bottom')
        .append('rect')
        .attr('x', 1) // Give 1 to hide the left starting point because it starts at 0 point.
        .attr('y', yScale(this.threshold))
        .attr('rx', 0)
        .attr('ry', 0)
        .attr('height', this.maxData)
        .attr('width', this.maxWidth - 2); // Subtract 2 to hide the part that converges to 0 at the end point.

        draw.append('path')
        .data([this.data])
        .attr('d', setBottomLine)
        .attr('clip-path', 'url(#' + this.elementId + '-bottom)')
        .attr('stroke', this.bottomStrokeColor)
        .attr('stroke-width', 1)
        .attr('fill', this.bottomFillColor);
    }
}
```

Complete.
