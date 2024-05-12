<script lang="ts">
    import { onMount } from "svelte";
    
	export let col: number;
	export let row: number;
	export let disabled_seats: string[];
    export let is_started_randomizing: boolean;

    let number_of_students = col * row - disabled_seats.length;

    // remove the seats over the size of the class
    $: col, row, (() => {
        let seats_to_remove: number[] = []
        disabled_seats.forEach((value, index) => {
            const [_x, _y] = value.split("|")
            const x = Number(_x);
            const y = Number(_y);

            if (x < 0 || x >= col || y < 0 || y >= row) {
                seats_to_remove.push(index);
            }
        })
        seats_to_remove.forEach(value => {
            disabled_seats.splice(value, 1);
        })
        disabled_seats = disabled_seats;
    })();

    $: number_of_students = col * row - disabled_seats.length;

    $: localStorage.setItem("col", col.toString());
    $: localStorage.setItem("row", row.toString());
    $: localStorage.setItem("disabled_seats", JSON.stringify(disabled_seats));

	function toggleSeat(event: any) {
		const x = event.target.dataset.x;
		const y = event.target.dataset.y;
		const disabled_index = disabled_seats.indexOf(`${x}|${y}`);
		if (disabled_index === -1) {
			disabled_seats.push(`${x}|${y}`);
		} else {
			disabled_seats.splice(disabled_index, 1);
		}
		disabled_seats = disabled_seats;
	}

    function start() {
        is_started_randomizing = is_started_randomizing ? false : true;
    }
    
	onMount(() => {
		window.onbeforeunload = null;
	})
</script>

<section>
    <p>학급 자리의 행과 열의 최댓값를 입력해주세요.</p>
    <div class="setting">
        <label>
            행: {col}
            <input type="range" min="1" max="10" step="1" bind:value={col} />
        </label>
        <label>
            열: {row}
            <input type="range" min="1" max="10" step="1" bind:value={row} />
        </label>
        <p>전체 학생 수: {number_of_students}</p>
    </div>


    <p>자리를 비우려면 그 자리를 클릭하세요.</p>
    <table >
        {#each [...Array(row).keys()] as y}
            <tr>
                {#each [...Array(col).keys()] as x}
                    {#if disabled_seats.includes(`${x}|${y}`)}
                        <td class="disabled" data-x={x} data-y={y} on:click={toggleSeat}></td>
                    {:else}
                        <td data-x={x} data-y={y} on:click={toggleSeat}><span></span></td>
                    {/if}
                {/each}
            </tr>
        {/each}
    </table>
    <button on:click={start} class="submit">무작위 배치 시작</button>
</section>

<style>
    .setting {
		display: flex;
		flex-direction: column;
		padding-bottom: 20px;
	}

	.setting label {
		display: flex;
		flex-direction: column;
		justify-content: center;
		margin-bottom: 10px;
	}

	.setting input {
		margin-left: 5px;
		width: 200px;
	}

	section {
		display: flex;
		flex-direction: column;
		align-items: center;
        padding: 0 20px;
	}

	table {
		margin-bottom: 30px;
	}

	table, td {
		border: 6px solid white;
  		border-collapse: collapse;
	}

	td {
		width: 100px;
		height: 60px;
		background: rgb(230, 230, 230);
		border-radius: 8px;
		cursor: pointer;
	}

	.disabled {
		background-color: rgb(179, 179, 179);
	}

	.submit {
		border: none;
		padding: 10px 20px;
		background-color: #62b87c;
		color: white;
		border-radius: 10px;
		cursor: pointer;
	}
</style>