<script>
  import { onMount } from "svelte";
  import ForceGraph3D from "3d-force-graph";
  import * as THREE from "three";
  import SpriteText from "three-spritetext";
  let target;
  onMount(async () => {
    let res = await fetch("/data.json");
    let data = await res.json();

    let graph = ForceGraph3D();
    graph(target)
      .graphData(data)
      .nodeThreeObject(node => {
        // use a sphere as a drag handle
        const obj = new THREE.Mesh(
          new THREE.SphereGeometry(10),
          new THREE.MeshBasicMaterial({
            depthWrite: false,
            transparent: true,
            opacity: 0
          })
        );

        // add text sprite as child
        const sprite = new SpriteText(node.name);
        sprite.color = node.color;
        sprite.textHeight = node.textHeight;
        obj.add(sprite);

        return obj;
      });
  });
</script>

<div bind:this={target} />
