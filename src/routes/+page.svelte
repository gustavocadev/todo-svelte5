<script lang="ts">
	type Todo = {
		text: string;
		done: boolean;
	};

	type Filters = 'all' | 'active' | 'completed';

	let todos = $state<Todo[]>([
		{
			text: 'Learn Svelte',
			done: true
		},
		{
			text: 'Learn SvelteKit',
			done: false
		}
	]);

	const filterTodos = () => {
		switch (filter) {
			case 'active':
				return todos.filter((todo) => !todo.done);
			case 'completed':
				return todos.filter((todo) => todo.done);
			case 'all':
				return todos;

			default:
				return [];
		}
	};

	let filter = $state<Filters>('all');
	let filteredTodos = $derived<Todo[]>(filterTodos());

	$effect(() => {
		const savedTodos = localStorage.getItem('todos');
		if (savedTodos) {
			todos = JSON.parse(savedTodos);
		}
	});

	$effect(() => {
		localStorage.setItem('todos', JSON.stringify(todos));
	});

	const addTodo = (event: KeyboardEvent) => {
		// if the key pressed is not the enter key, return
		if (event.key !== 'Enter') return;

		const todoEl = event.target as HTMLInputElement;
		const text = todoEl.value.trim();
		const done = false;

		todos = [...todos, { text, done }];

		todoEl.value = '';
	};

	const editTodo = (event: Event) => {
		const inputEl = event.target as HTMLInputElement;

		// lets capture the index of the todo
		const index = Number(inputEl.dataset['index']);

		todos[index].text = inputEl.value;
	};

	const toggleTodo = (event: Event) => {
		const inputEl = event.target as HTMLInputElement;

		// lets capture the index of the todo
		const index = Number(inputEl.dataset['index']);

		todos[index].done = !todos[index].done;
	};

	const setFilter = (newFilter: Filters) => {
		filter = newFilter;
	};

	const remaining = () => {
		return todos.filter((todo) => !todo.done).length;
	};
</script>

<main class="flex flex-col items-center justify-center gap-4 p-4">
	<input
		type="text"
		class="input input-bordered w-full max-w-xs"
		onkeydown={addTodo}
		placeholder="Add a new todo"
	/>

	{#each filteredTodos as todo, idx}
		<section class="flex flex-row items-center gap-4" class:completed={todo.done}>
			<input
				type="text"
				bind:value={todo.text}
				class="input input-bordered w-full max-w-xs"
				oninput={editTodo}
				data-index={idx}
			/>
			<input
				type="checkbox"
				checked={todo.done}
				class="checkbox"
				onchange={toggleTodo}
				data-index={idx}
			/>
		</section>
	{/each}

	<div class="filters">
		<!-- {#each ['all', 'active', 'completed'] as filter}
			<button onclick={() => setFilter(filter)}>{filter}</button>
		{/each} -->
		<button onclick={() => setFilter('all')} class="btn btn-primary">All</button>
		<button onclick={() => setFilter('active')} class="btn btn-primary">Active</button>
		<button onclick={() => setFilter('completed')} class="btn btn-primary">Completed</button>
	</div>

	<p>{remaining()} remaining</p>
</main>
