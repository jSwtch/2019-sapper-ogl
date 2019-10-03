    <script >
    import {onMount} from "svelte"
    import {Renderer, Geometry, Program, Mesh, Camera, Transform, Texture} from 'ogl/src/Core.js';
    import {Sphere, Orbit} from 'ogl/src/Extras.js';

    let elOGL;

    onMount(()=> {
               
        const vertex = `
            precision highp float;
            precision highp int;

            attribute vec2 uv;
            attribute vec3 position;
            attribute vec3 normal;

            uniform mat4 modelViewMatrix;
            uniform mat4 projectionMatrix;

            varying vec2 vUv;

            void main() {
                vUv = uv;
                gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
            }
        `;

        const fragment = `
            precision highp float;
            precision highp int;

            uniform sampler2D tMap;

            varying vec2 vUv;

            void main() {
                vec3 tex = texture2D(tMap, vUv).rgb;
                
                gl_FragColor.rgb = tex;
                gl_FragColor.a = 1.0;
            }
        `;

        {
            const renderer = new Renderer({dpr: 2});
            const gl = renderer.gl;
            elOGL.appendChild(gl.canvas);

            gl.clearColor(1, 1, 1, 1);

            const camera = new Camera(gl, {fov: 45});
            camera.position.set(0, 0, 8);

            const controls = new Orbit(camera, {
                enablePan: false,
                enableZoom: false,
            });

            function resize() {
                renderer.setSize(window.innerWidth, window.innerHeight);
                camera.perspective({aspect: gl.canvas.width / gl.canvas.height});
            }
            window.addEventListener('resize', resize, false);
            resize();

            const scene = new Transform();

            // Texture is equirectangular
            const texture = new Texture(gl);
            const img = new Image();
            img.onload = () => texture.image = img;
            // img.src = 'assets/sky.jpg';
			// img.src = './satara_night_1k.hdr';
			// img.src = './topsoliddesigndevelopment42080.jpg';
			// img.src = './space.jpg';
			img.src = './test2k.jpg';


            // Use Sphere geometry to render equirectangular textures
            const geometry = new Sphere(gl, {radius: 1, widthSegments: 64});

            const program = new Program(gl, {
                vertex,
                fragment,
                uniforms: {
                    tMap: {value: texture},
                },

                // Need inside of sphere to be visible
                cullFace: null,
            });

            // A smaller sphere in the center just to help illustrate
            const mesh = new Mesh(gl, {geometry, program});
            mesh.setParent(scene);

            // Camera will dwell inside skybox
            const skybox = new Mesh(gl, {geometry, program});
            skybox.scale.set(10);
            skybox.setParent(scene);

            requestAnimationFrame(update);
            function update() {
                requestAnimationFrame(update);

                controls.update();
                renderer.render({scene, camera});
            }
        }
    })

</script>

<style>

</style>

<svelte:head>
	<title>HOME</title>
</svelte:head>

<h1>Hello from index</h1>
<div bind:this={elOGL}></div>
