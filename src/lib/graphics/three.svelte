<script>
	import { onMount, onDestroy  } from 'svelte';
	import { screenType } from '$lib/store/store';
	import { page } from '$app/stores';
	import { afterNavigate } from '$app/navigation';

	import * as THREE from 'three';
	import Stats from 'stats.js'

	import vertexShader from './shaders/vertexShader-three.glsl';
	import fragmentShader_aufbau from './shaders/fragmentShader-aufbau.glsl';
	import fragmentShader_niels from './shaders/fragmentShader-niels.glsl';
	import fragmentShader_raum from './shaders/fragmentShader-raum.glsl';

	let shaderMaterial_aufbau, shaderMaterial_niels, shaderMaterial_raum;

	let container;
	let stats;

	let camera, scene, renderer;

	let width = window.innerWidth;
	let height = window.innerHeight;

	let mouse = new THREE.Vector2();
	const clock = new THREE.Clock();

	stats = new Stats()
	stats.showPanel(0) // 0: fps, 1: ms, 2: mb, 3+: custom
	document.body.appendChild(stats.dom)
	
	init();
	animate();

	function setupShaderMaterials() {
		const uniformsBase = {
			time: { value: 0 },
			mouse: { value: mouse }
		};

		const colors = {
			color1: new THREE.Color(0xd0d0d0),
			color2: new THREE.Color(0xbb4500),
			color3: new THREE.Color(0xdaaa55),
			color4: new THREE.Color(0x006994 ),
			color5: new THREE.Color(0x5099b4 ),
			color6: new THREE.Color(0x0000ff),
			color7: new THREE.Color(0x00ff00),
			color9: new THREE.Color(0x8fbd5a),
			color0: new THREE.Color(0x232323),
		}

		shaderMaterial_aufbau = new THREE.ShaderMaterial({
			vertexShader: vertexShader,
			fragmentShader: fragmentShader_aufbau,
			uniforms: {
				...uniformsBase,
				color1: { value: colors.color1 },
				color2: { value: colors.color5 },
				color3: { value: colors.color9 },
			}
		});

		shaderMaterial_niels = new THREE.ShaderMaterial({
			vertexShader: vertexShader,
			fragmentShader: fragmentShader_niels,
			uniforms: {
				...uniformsBase,
				color1: { value: colors.color1 },
				color2: { value: colors.color2 },
				color3: { value: colors.color6 },
				color4: { value: colors.color7 },
			}
		});

		shaderMaterial_raum = new THREE.ShaderMaterial({
			vertexShader: vertexShader,
			fragmentShader: fragmentShader_raum,
			uniforms: {
				...uniformsBase,
				color1: { value: colors.color1 },
				color2: { value: colors.color1 },
				color3: { value: colors.color0 },
			}
		});
	}

	function updateShaderUniforms() {
		const elapsedTime = clock.getElapsedTime();

		if ($page.url.pathname == '/') {
			if ($screenType == 1) {
				shaderMaterial_aufbau.uniforms.time.value = elapsedTime;
				shaderMaterial_aufbau.uniforms.mouse.value = {
					x: 4 * Math.cos(elapsedTime * 0.1),
					y: 5 * Math.cos(elapsedTime * 0.1)
				} 
			} else {
				shaderMaterial_aufbau.uniforms.mouse.value = mouse;
				shaderMaterial_aufbau.uniforms.time.value = elapsedTime ;
			}
		}

		if ($page.url.pathname == '/ripple') {
			if ($screenType == 1) {
				shaderMaterial_niels.uniforms.mouse.value = {
					x: clock.getElapsedTime() * 1,
					y: clock.getElapsedTime() * 0.1
				};
			} else {
				shaderMaterial_niels.uniforms.mouse.value = {
					x: mouse.x + clock.getElapsedTime() * 0.1,
					y: mouse.y + clock.getElapsedTime() * 1
				};
			}
		}

		if ($page.url.pathname == '/tangent') {
			shaderMaterial_raum.uniforms.mouse.value = mouse;
			shaderMaterial_raum.uniforms.time.value = elapsedTime;
		}
	}

	function init() {
		camera = new THREE.PerspectiveCamera(20, width / height, 1, 800);
		camera.position.z = 400;

		scene = new THREE.Scene();
		scene.background = new THREE.Color(0x232323);

		setupShaderMaterials();
		setScene();

		renderer = new THREE.WebGLRenderer({ antialias: false });
		renderer.setPixelRatio(window.devicePixelRatio);
		renderer.setSize(width, height);

		onMount(() => {
			container.appendChild(renderer.domElement);


		});

		window.addEventListener('mousemove', onDocumentMouseMove);
		window.addEventListener('resize', onWindowResize);
		// window.addEventListener('navigate', onNavigate);
	}

	function setHome () {

		let plane4 = new THREE.Mesh(new THREE.PlaneGeometry(1000, 1000), shaderMaterial_aufbau);
		let plane5 = new THREE.Mesh(new THREE.PlaneGeometry(100, 100), shaderMaterial_aufbau);
		scene.add(plane4);

		if ($screenType != 1) {
			plane5.position.z = 200;
			scene.add(plane5);

		} else {
			plane5.position.z = 100;
			plane5.rotation.z = Math.PI / 2
			scene.add(plane5);
		}
	}

	function setNiels () {
		let plane3 = new THREE.Mesh(new THREE.PlaneGeometry(600, 600), shaderMaterial_niels);
		plane3.position.z = -0.1;
		scene.add(plane3)
	}

	function setSicovecas () {
		let plane = new THREE.Mesh(new THREE.PlaneGeometry(1000, 1000), shaderMaterial_raum);
		let plane2 = new THREE.Mesh(new THREE.PlaneGeometry(100, 100), shaderMaterial_raum);
		plane2.position.z = 200;
		scene.add(plane, plane2);
	}

	function setScene () {

		if ($page.url.pathname == '/') {
			setHome();
		}

		if ($page.url.pathname == '/ripple') {
			setNiels();
		}

		if ($page.url.pathname == '/tangent') {
			setSicovecas();
		}
	}

	afterNavigate (onNavigate);
	function onNavigate() {

		for( var i = scene.children.length - 1; i >= 0; i--) { 
				let obj = scene.children[i];
				scene.remove(obj); 
		}

		setScene();

	}

	function onWindowResize() {
		let width = window.innerWidth;
		let height = window.innerHeight;

		camera.aspect = width / height;
		camera.updateProjectionMatrix();

		renderer.setSize(width, height);
	}

	function onDocumentMouseMove(event) {
    var clientX = event.clientX;
    var clientY = event.clientY;

    mouse.x = (clientX / window.innerWidth) * 2 - 1;
		mouse.y = -(clientY / window.innerHeight) * 2 + 1;

	};

	function animate() {
		requestAnimationFrame(animate);
		stats.begin()
		render();
		stats.end()
	}

	function render() {
		updateShaderUniforms();
		renderer.render(scene, camera);
	}

	onDestroy(() => {
    // Remove event listeners
    window.removeEventListener('mousemove', onDocumentMouseMove);
    window.removeEventListener('resize', onWindowResize);
    // Dispose of the materials, geometries, textures etc.
    scene.traverse(object => {
        if (object instanceof THREE.Mesh) {
            object.geometry.dispose();
            object.material.dispose();
        }
        // Consider adding other types if they hold resources (like textures).
    });
    
    // If you've added any textures, dispose of them too
    // texture.dispose();

    renderer.dispose();  // Dispose of the renderer's resources
});

</script>

<div bind:this={container} class:geometry={true} />

<style>
	.geometry {
		position: absolute;
		overflow: hidden;
		z-index: -1;
	}
</style>
