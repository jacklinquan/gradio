<script>
	import { tick } from "svelte";

	export let theme = "";
	export let label = "Title";
	export let headers = [];
	export let values = [
		["Frank", 32, "Male"],
		["Beatrice", 99, "Female"],
		["Simone", 999, "Male"]
	];
	export let setValue;
	export let editable = true;

	let id = 0;
	let editing = false;
	let selected = false;
	let els = {};

	function make_headers(_h) {
		if (_h.length === 0) {
			return values[0].map((_, i) => {
				const _id = ++id;
				els[_id] = { cell: null, input: null };
				return { id: _id, value: i + 1 };
			});
		} else {
			return _h.map((h) => {
				const _id = ++id;
				els[_id] = { cell: null, input: null };
				return { id: _id, value: h };
			});
		}
	}

	let _headers = make_headers(headers);

	let data = values.map((x) =>
		x.map((n) => {
			const _id = ++id;
			els[id] = { input: null, cell: null };
			return { value: n, id: _id };
		})
	) || [
		Array(headers.length)
			.fill(0)

			.map((_) => {
				const _id = ++id;
				els[id] = { input: null, cell: null };

				return { value: "", id: _id };
			})
	];

	$: setValue(data.map((r) => r.map(({ value }) => value)));

	function get_sort_status(name, sort) {
		if (!sort) return "none";
		if (sort[0] === name) {
			return sort[1];
		}
	}

	async function start_edit(id) {
		if (!editable) return;
		editing = id;
		await tick();
		const { input } = els[id];
		input.focus();
	}

	function handle_keydown(event, i, j, id) {
		let is_data;
		switch (event.key) {
			case "ArrowRight":
				if (editing) break;
				event.preventDefault();
				is_data = data[i][j + 1];
				selected = is_data ? is_data.id : selected;
				break;
			case "ArrowLeft":
				if (editing) break;
				event.preventDefault();
				is_data = data[i][j - 1];
				selected = is_data ? is_data.id : selected;
				break;
			case "ArrowDown":
				if (editing) break;
				event.preventDefault();
				is_data = data[i + 1];
				selected = is_data ? is_data[j].id : selected;
				break;
			case "ArrowUp":
				if (editing) break;
				event.preventDefault();
				is_data = data[i - 1];
				selected = is_data ? is_data[j].id : selected;
				break;
			case "Escape":
				if (!editable) break;
				event.preventDefault();
				editing = false;
				break;
			case "Enter":
				if (!editable) break;
				event.preventDefault();
				if (editing === id) {
					editing = false;
				} else {
					editing = id;
				}
				break;
			default:
				break;
		}
	}

	async function handle_cell_click(id) {
		editing = false;
		selected = id;
	}

	async function set_focus(id, type) {
		if (type === "edit" && typeof id == "number") {
			await tick();
			els[id].input.focus();
		}

		if (type === "edit" && typeof id == "boolean") {
			let cell = els[selected]?.cell;
			await tick();
			cell?.focus();
		}

		if (type === "select" && typeof id == "number") {
			const { cell } = els[id];
			cell.setAttribute("tabindex", 0);
			await tick();
			els[id].cell.focus();
		}
	}

	$: set_focus(editing, "edit");
	$: set_focus(selected, "select");

	let sort_direction;
	let sort_by;

	function sort(col, dir) {
		if (dir === "asc") {
			data = data.sort((a, b) => (a[col].value < b[col].value ? -1 : 1));
		} else if (dir === "des") {
			data = data.sort((a, b) => (a[col].value > b[col].value ? -1 : 1));
		}
	}

	function handle_sort(col) {
		if (typeof sort_by !== "number" || sort_by !== col) {
			sort_direction = "asc";
			sort_by = col;
		} else {
			if (sort_direction === "asc") {
				sort_direction = "des";
			} else if (sort_direction === "des") {
				sort_direction = "asc";
			}
		}

		sort(col, sort_direction);
	}

	let header_edit;

	async function edit_header(_id, select) {
		if (!editable) return;
		header_edit = _id;
		await tick();
		els[_id].input.focus();

		if (select) els[_id].input.select();
	}

	function end_header_edit(event) {
		if (!editable) return;

		switch (event.key) {
			case "Escape":
				event.preventDefault();
				header_edit = false;
				break;
			case "Enter":
				event.preventDefault();
				header_edit = false;
		}
	}

	function add_row() {
		data.push(
			headers.map(() => {
				const _id = ++id;
				els[_id] = { cell: null, input: null };
				return { id: _id, value: "" };
			})
		);
		data = data;
	}

	async function add_col() {
		for (let i = 0; i < data.length; i++) {
			const _id = ++id;
			els[_id] = { cell: null, input: null };
			data[i].push({ id: _id, value: "" });
		}

		const _id = ++id;
		els[_id] = { cell: null, input: null };
		_headers.push({ id: _id, value: `Header ${_headers.length + 1}` });

		data = data;
		_headers = _headers;

		await tick();

		edit_header(_id, true);
	}

	const double_click = (node, { click, dblclick }) => {
		let timer;

		function handler(event) {
			if (timer) {
				clearTimeout(timer);
				timer = undefined;
				dblclick(event);
			} else {
				timer = setTimeout(() => {
					click(event);
					timer = undefined;
				}, 250);
			}
		}

		node.addEventListener("click", handler);

		return {
			destroy: () => node.removeEventListener("click", handler)
		};
	};
</script>

<h4 id="title">{label}</h4>

<div class="shadow overflow-hidden border-gray-200 rounded-sm relative">
	<table
		id="grid"
		role="grid"
		aria-labelledby="title"
		class="min-w-full divide-y divide-gray-200 "
	>
		<thead class="bg-gray-50">
			<tr>
				{#each _headers as { value, id }, i (id)}
					<th
						use:double_click={{
							click: () => handle_sort(i),
							dblclick: () => edit_header(id)
						}}
						aria-sort={get_sort_status(value, sort_by)}
						class="relative after:absolute after:opacity-0 after:content-['▲'] after:ml-2 after:inset-y-0 after:h-[1.05rem] after:m-auto relative px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
						class:sorted={sort_by === i}
						class:des={sort_by === i && sort_direction === "des"}
					>
						{#if header_edit === id}
							<input
								class="bg-transparent inset-y-0 left-6 w-full outline-none absolute p-0 w-3/4 text-xs font-medium text-gray-500 uppercase tracking-wider"
								tabindex="-1"
								bind:value
								bind:this={els[id].input}
								on:keydown={end_header_edit}
								on:blur={({ currentTarget }) =>
									currentTarget.setAttribute("tabindex", -1)}
							/>
						{/if}
						<span
							tabindex="-1"
							role="button"
							class="min-h-full"
							class:opacity-0={header_edit === id}>{value}</span
						>
					</th>
				{/each}
			</tr></thead
		><tbody class="bg-white divide-y divide-gray-200">
			{#each data as row, i (row)}
				<tr>
					{#each row as { value, id }, j (id)}
						<td
							tabindex="-1"
							class="p-0 whitespace-nowrap display-block outline-none relative "
							on:dblclick={() => start_edit(id)}
							on:click={() => handle_cell_click(id)}
							on:keydown={(e) => handle_keydown(e, i, j, id)}
							bind:this={els[id].cell}
							on:blur={({ currentTarget }) =>
								currentTarget.setAttribute("tabindex", -1)}
						>
							<div
								class:border-gray-600={selected === id}
								class:border-transparent={selected !== id}
								class="min-h-[3.3rem] px-5 py-3  border-[0.125rem]"
							>
								{#if editing === id}
									<input
										class="w-full outline-none absolute p-0 w-3/4"
										tabindex="-1"
										bind:value
										bind:this={els[id].input}
										on:blur={({ currentTarget }) =>
											currentTarget.setAttribute("tabindex", -1)}
									/>
								{/if}
								<span
									class=" cursor-default w-full"
									class:opacity-0={editing === id}
									tabindex="-1"
									role="button"
								>
									{value}
								</span>
							</div>
						</td>
					{/each}
				</tr>
			{/each}
		</tbody>
	</table>
</div>
{#if editable}
	<div class="flex justify-end ">
		<button
			on:click={add_col}
			class="hover:bg-gray-100 dark:hover:bg-gray-600 shadow  py-1 px-3 rounded transition  focus:outline-none m-2 mr-0"
			>New Column</button
		>
		<button
			on:click={add_row}
			class="bg-yellow-500 hover:bg-yellow-400 dark:bg-red-700 dark:hover:bg-red-600 text-white shadow py-1 px-3 rounded transition focus:outline-none m-2 mr-0"
			>New Row</button
		>
	</div>
{/if}

<style>
	.sorted::after {
		opacity: 1;
	}

	.des::after {
		transform: rotate(180deg) translateY(1.5px);
	}
</style>
