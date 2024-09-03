<template>
	<div class="row">
		<div class="selects">
			<span>Total: {{ totalTodos }}</span>
			<div>
				<span>Filter by:</span>
				<select v-model="filter">
					<option value="all">All</option>
					<option value="completed">Completed</option>
					<option value="uncompleted">Uncompleted</option>
				</select>
			</div>
			<div>
				<span>limit:</span>
				<select v-model="limitTodo">
					<option value="10">10</option>
					<option value="15">15</option>
					<option value="20">20</option>
				</select>
			</div>
		</div>
		<div class="to-do__wrap">
			<div class="to-do__title">
				<h1>To-do list</h1>
				<img
					src="../public/images/icons/list.png"
					alt="list-icon"
				/>
			</div>
			<form
				class="to-do__form"
				name="to-do__form"
				@submit.prevent="addTodo"
			>
				<input
					class="to-do__enter"
					type="text"
					placeholder="Add your text"
					name="to-do__enter"
					v-model="newTodo"
				/>
				<button class="to-do__button">Add</button>
			</form>
			<div class="to-do__list-wrap">
				<div
					v-if="loading"
					class="loader"
				></div>
				<ul class="to-do__list">
					<li
						class="to-do__item"
						v-for="todo in filteredTodos"
						:key="todo.id"
					>
						<input
							type="checkbox"
							:checked="todo.completed"
							:id="`checkbox-${todo.id}`"
						/>
						<label
							v-if="!todo.isEditing"
							:for="`checkbox-${todo.id}`"
							@dblclick="editTodo(todo)"
							>{{ todo.title }}</label
						>
						<input
							v-else
							type="text"
							v-model="todo.title"
							@blur="doneEdit(todo)"
							@keyup.enter="doneEdit(todo)"
						/>
						<img
							class="to-do__del"
							@click="deleteTodo(todo.id)"
							src="../public/images/icons/delete.svg"
							alt="delete icon"
						/>
					</li>
				</ul>
			</div>
		</div>
	</div>
</template>

<script>
import axios from 'axios';

export default {
	data() {
		return {
			delay: () => new Promise(resolve => setTimeout(resolve, 2000)),
			baseUrl: 'https://jsonplaceholder.typicode.com/',
			toDoList: [],
			newTodo: '',
			loading: false,
			limitTodo: 10,
			filter: 'all',
			totalTodos: 0,
		};
	},
	computed: {
		filteredTodos() {
			if (this.filter === 'completed') {
				return this.toDoList.filter(todo => todo.completed);
			} else if (this.filter === 'uncompleted') {
				return this.toDoList.filter(todo => !todo.completed);
			} else {
				return this.toDoList;
			}
		},
	},
	watch: {
		limitTodo() {
			this.getToDos();
		},
	},
	methods: {
		async getToDos() {
			try {
				this.loading = true;
				await this.delay();
				const { data } = await axios.get(`${this.baseUrl}todos`, {
					params: { _limit: this.limitTodo },
				});

				this.toDoList = data.map(todo => ({ ...todo, isEditing: false }));
			} catch (error) {
				console.error('error', error);
			} finally {
				this.loading = false;
			}
		},
		async getTotalToDosCount() {
			try {
				const { data } = await axios.get(`${this.baseUrl}todos`);
				this.totalTodos = data.length;
			} catch (error) {
				console.error('error', error);
			}
		},
		deleteTodo(id) {
			this.toDoList = this.toDoList.filter(todo => todo.id !== id);
			this.totalTodos--;
		},
		addTodo() {
			if (!this.newTodo.trim()) return;
			this.toDoList.unshift({
				id: Date.now(),
				title: this.newTodo,
				completed: false,
				isEditing: false,
			});
			this.newTodo = '';
			this.totalTodos++;
		},
		editTodo(todo) {
			todo.isEditing = true;
		},
		doneEdit(todo) {
			if (!todo.title.trim()) {
				this.deleteTodo(todo.id);
			} else {
				todo.isEditing = false;
			}
		},
	},
	mounted() {
		this.getToDos();
		this.getTotalToDosCount();
	},
};
</script>

<style scoped lang="scss">
.row {
	padding: 50px;
}
.selects {
	display: flex;
	flex-direction: column;
	justify-content: center;
	align-items: center;
	gap: 10px;
	margin-bottom: 25px;
	span {
		margin-right: 10px;
		font-size: 20px;
	}
	select {
		padding: 3px;
		border-radius: 5px;
		cursor: pointer;
	}
}
.loader {
	margin: 0 auto;
	width: 50px;
	padding: 8px;
	aspect-ratio: 1;
	border-radius: 50%;
	background: #25b09b;
	--_m: conic-gradient(#0000 10%, #000), linear-gradient(#000 0 0) content-box;
	-webkit-mask: var(--_m);
	mask: var(--_m);
	-webkit-mask-composite: source-out;
	mask-composite: subtract;
	animation: l3 2s infinite linear;
}
@keyframes l3 {
	to {
		transform: rotate(1turn);
	}
}
.to-do {
	// .to-do__wrap
	&__wrap {
		background-color: white;
		border-radius: 10px;
		max-width: 600px;
		padding: 35px;
		margin: 0 auto;
		box-shadow: 0px 0px 18px 0px rgba(0, 0, 0, 0.75);
	}

	// .to-do__title
	&__title {
		display: flex;
		align-items: center;
		gap: 10px;
		margin-bottom: 30px;
		h1 {
			font-weight: 800;
			text-transform: capitalize;
			font-size: 30px;
			color: rgb(32, 19, 110);
		}
		img {
			width: 30px;
			height: 30px;
		}
	}
	// .to-do__form
	&__form {
		display: flex;
		margin-bottom: 30px;
	}
	// .to-do__enter
	&__enter {
		background-color: #dee8f1;
		border-radius: 30px;
		width: 81%;
		padding: 20px 60px 20px 20px;
		border: none;
	}
	// .to-do__button
	&__button {
		border-radius: 30px;
		width: 150px;
		color: white;
		background-color: rgb(255, 123, 0);
		margin-left: -50px;
		cursor: pointer;
		transition: 0.3s;
		border: 2px solid rgb(255, 123, 0);

		&:hover {
			background-color: white;
			color: black;
		}
	}
	// .to-do__list
	&__list {
		display: flex;
		flex-direction: column;
		gap: 20px;
	}

	// .to-do__item
	&__item {
		display: flex;
		justify-content: space-between;
		align-items: center;
		input[type='checkbox'] {
			display: none;
			&:checked + label {
				text-decoration: line-through;
				color: #fb771a;
				&::before {
					border-color: #fb771a;
				}
				&::after {
					content: '';
					position: absolute;
					border: solid #fb771a;
					border-width: 0 2px 2px 0;
					width: 7px;
					height: 10px;
					left: 8px;
					top: 50%;
					transform: rotate(45deg) translate(-50%, -50%);
					transition: 0.5s;
				}
			}
		}
		label {
			position: relative;
			padding-left: 35px;
			transition: 0.3s;
			word-wrap: break-word;
			overflow-x:hidden &:hover {
				color: rgb(89, 8, 228);
			}
			cursor: pointer;
			&::before {
				content: '';
				position: absolute;
				border: 1px solid rgb(102, 100, 100);
				border-radius: 50%;
				width: 25px;
				height: 25px;
				left: 0;
				top: 50%;
				transform: translateY(-50%);
				transition: 0.3s;
			}
		}
		input[type='text'] {
			border-radius: 10px;
			padding: 2px 10px;
			border-color: #fb771a;
			width: 95%;
		}
	}
	// .to-do__del
	&__del {
		width: 12px;
		height: 12px;
		cursor: pointer;
		transition: 0.3s;
		&:hover {
			filter: invert(49%) sepia(55%) saturate(1466%) hue-rotate(349deg)
				brightness(101%) contrast(97%);
		}
	}
}
</style>
