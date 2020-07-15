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
    graph.d3Force("charge").strength(-300);
    graph(target)
      .graphData(data)
      .backgroundColor("#E8E9ED")
      .linkColor(() => "black")
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
      })
      .onNodeHover(node => (target.style.cursor = node ? "pointer" : null))
      .onNodeClick(node => {
        // Aim at node from outside it
        const distance = 40;
        const distRatio = 1 + distance / Math.hypot(node.x, node.y, node.z);

        graph.cameraPosition(
          {
            x: node.x * distRatio,
            y: node.y * distRatio,
            z: node.z * distRatio * 4
          }, // new position
          node, // lookAt ({ x, y, z })
          3000 // ms transition duration
        );
      });
  });
</script>

<div bind:this={target} />
