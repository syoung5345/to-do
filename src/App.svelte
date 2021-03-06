<script>
  import { onMount } from "svelte";
	import { quintOut } from 'svelte/easing';
	import { crossfade } from 'svelte/transition';
	import { flip } from 'svelte/animate';
  export let date;

  onMount(async () => {
    const res = await fetch("/api/date");
    const newDate = await res.text();
    date = newDate;
  });
  
	const [send, receive] = crossfade({
		fallback(node, params) {
			const style = getComputedStyle(node);
			const transform = style.transform === 'none' ? '' : style.transform;

			return {
				duration: 600,
				easing: quintOut,
				css: t => `
					transform: ${transform} scale(${t});
					opacity: ${t}
				`
			};
		}
	});

	let todos = [];

	let uid = todos.length + 1;

	function add(input) {
		const todo = {
			id: uid++,
			done: false,
			description: input.value
		};

		todos = [todo, ...todos];
		input.value = '';
	}

	function remove(todo) {
		todos = todos.filter(t => t !== todo);
	}
</script>

<style>
	.new-todo {
		font-size: 1.4em;
		width: 100%;
		margin: 2em 0 1em 0;
	}

	.board {
		max-width: 40em;
		margin: 0 auto;
	}

	.left, .right {
		float: left;
		width: 50%;
		padding: 0 1em 0 0;
		box-sizing: border-box;
	}

	h2 {
		font-size: 2em;
		font-weight: 200;
		user-select: none;
	}

	label {
		top: 0;
		left: 0;
		display: block;
		font-size: 1em;
		line-height: 1;
		padding: 0.5em;
		margin: 0 auto 0.5em auto;
		border-radius: 1rem;
		background-color: #eee;
		user-select: none;
	}

	input { 
    margin: 0;
    font-size: 1em;
    color: gray;
    padding-left: .5rem;
  }

	.right label {
		background-color: rgb(180,240,100);
	}

	button {
		float: right;
		height: 1em;
		box-sizing: border-box;
		padding: 0 0.5em;
		line-height: 1;
		background-color: transparent;
		border: none;
		color: rgb(170,30,30);
		opacity: 0;
		transition: opacity 0.2s;
	}

	label:hover button {
		opacity: 1;
	}
</style>

<main>
  <h1>todo</h1>
  <div class='board'>
    <input
      class="new-todo"
      placeholder="what needs to be done?"
      on:keydown="{event => event.which === 13 && add(event.target)}"
    >

    <div class='left'>
      <h2>todo</h2>
      {#each todos.filter(t => !t.done) as todo (todo.id)}
        <label
          in:receive="{{key: todo.id}}"
          out:send="{{key: todo.id}}"
          animate:flip
        >
          <input type=checkbox bind:checked={todo.done}>
          {todo.description}
          <button on:click="{() => remove(todo)}">x</button>
        </label>
      {/each}
    </div>

    <div class='right'>
      <h2>done</h2>
      {#each todos.filter(t => t.done) as todo (todo.id)}
        <label
          in:receive="{{key: todo.id}}"
          out:send="{{key: todo.id}}"
          animate:flip
        >
          <input type=checkbox bind:checked={todo.done}>
          {todo.description}
          <button on:click="{() => remove(todo)}">x</button>
        </label>
      {/each}
    </div>
  </div>
  
  <h2>Svelte + Node.js API</h2>
  <p>
    <a
      href="https://github.com/zeit/now/tree/master/examples/svelte"
      target="_blank"
      rel="noreferrer noopener">
      This project
    </a>
    is a
    <a href="https://svelte.dev/">Svelte</a>
    app with three directories,
    <code>/public</code>
    for static assets,
    <code>/src</code>
    for components and content, and
    <code>/api</code>
    which contains a serverless
    <a href="https://nodejs.org/en/">Node.js</a>
    function. See
    <a href="/api/date">
      <code>api/date</code>
      for the Date API with Node.js
    </a>
    .
  </p>
  <br />
  <h2>The date according to Node.js is:</h2>
  <p>{date ? date : 'Loading date...'}</p>
</main>
