<script>
  import { onMount } from "svelte";
  import ForceGraph3D from "3d-force-graph";
  import { Mesh, SphereGeometry, MeshBasicMaterial } from "three";
  import SpriteText from "three-spritetext";
  import { Circle } from "svelte-loading-spinners";
  let target;
  let loading = true;
  onMount(async () => {
    let res = await fetch("/assets/data.json");
    let data = await res.json();
    let counter = 0;

    let graph = ForceGraph3D();
    graph.d3Force("charge").strength(-300);
    graph(target)
      .graphData(data)
      .backgroundColor("#E8E9ED")
      .linkColor(() => "#A8A8A8")
      .nodeThreeObject(node => {
        // use a sphere as a drag handle
        const obj = new Mesh(
          new SphereGeometry(10),
          new MeshBasicMaterial({
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
        const distance = 200;
        const distRatio = 1 + distance / Math.hypot(node.x, node.y, node.z);

        graph.cameraPosition(
          {
            x: node.x * distRatio,
            y: node.y * distRatio,
            z: node.z * distRatio
          }, // new position
          node, // lookAt ({ x, y, z })
          3000 // ms transition duration
        );
        setTimeout(() => {
          window.open(`https://www.w3schools.com/tags/ref_byfunc.asp`);
        }, 3000);
        //
      })
      .onEngineTick(() => {
        counter += 1;
        if (counter >= 200) {
          loading = false;
        }
      });
  });
</script>

<style>
  .loading {
    background-color: #e8e9ed;
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
  }
  * {
    font-family: sans-serif;
    margin-top: 15px;
  }
</style>

{#if loading}
  <div class="loading">
    <Circle size="60" unit="px" color="#E65127" />
    <h1>3D graph loading</h1>
    <p>Best view on desktop</p>
    <p>You might experience performance issue on mobile device</p>
  </div>
{/if}
<div bind:this={target} />
