<script lang="ts">
  import { Editor } from '@annotorious/annotorious/src';
  import { getMaskDimensions, ShapeType } from '@annotorious/annotorious';
  import type { Ellipse, Shape, Transform } from '@annotorious/annotorious';
  
  export let shape: Ellipse;
  export let computedStyle: string | undefined;
  export let transform: Transform;
  export let viewportScale: number = 1;
  export let svgEl: SVGSVGElement;

  $: geom = shape.geometry;

  const isEllipse = (shape: Shape): shape is Ellipse => {
    const { geometry } = shape;
    return shape.type === ShapeType.ELLIPSE &&
      'cx' in geometry && typeof geometry.cx === 'number' &&
      'cy' in geometry && typeof geometry.cy === 'number' &&
      'rx' in geometry && typeof geometry.rx === 'number' &&
      'ry' in geometry && typeof geometry.ry === 'number';
  };

  const editor = (shape: Shape, _handle: string, delta: [number, number]) => {
    if (!isEllipse(shape)) return shape;

    const { geometry } = shape;
    const { bounds } = geometry;

    const [dx, dy] = delta;

    return {
      ...shape,
      geometry: {
        ...geometry,
        cx: geometry.cx + dx,
        cy: geometry.cy + dy,
        bounds: {
          minX: bounds.minX + dx,
          minY: bounds.minY + dy,
          maxX: bounds.maxX + dx,
          maxY: bounds.maxY + dy
        }
      }
    };
  }

  $: mask = getMaskDimensions(geom.bounds, 2 / viewportScale);
  const maskId = `ellipse-mask-${Math.random().toString(36).substring(2, 12)}`;
</script>

<Editor
  shape={shape}
  transform={transform}
  editor={editor}
  svgEl={svgEl}
  on:grab
  on:change 
  on:release
  let:grab={grab}>
  <defs>
    <mask id={maskId} class="a9s-ellipse-editor-mask">
      <rect x={mask.x} y={mask.y} width={mask.w} height={mask.h} />
      <ellipse cx={geom.cx} cy={geom.cy} rx={geom.rx} ry={geom.ry} />
    </mask>
  </defs>

  <ellipse 
    class="a9s-outer"
    mask={`url(#${maskId})`}
    on:pointerdown={grab('SHAPE')}
    cx={geom.cx} cy={geom.cy} rx={geom.rx} ry={geom.ry} />

  <ellipse 
    class="a9s-inner a9s-shape-handle"
    style={computedStyle}
    on:pointerdown={grab('SHAPE')}
    cx={geom.cx} cy={geom.cy} rx={geom.rx} ry={geom.ry} />

  <circle
    cx={geom.cx}
    cy={geom.cy}
    r={2 / viewportScale}
    fill="white"
    stroke="black"
    stroke-width={1 / viewportScale}
    pointer-events="none" />
</Editor>

<style>
  mask.a9s-ellipse-editor-mask > rect {
    fill: #fff;
  }

  mask.a9s-ellipse-editor-mask > ellipse {
    fill: #000;
  }
</style>
