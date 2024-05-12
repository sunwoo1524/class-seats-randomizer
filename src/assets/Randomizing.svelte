<script lang="ts">
    import { onMount } from "svelte";
    import html2canvas from 'html2canvas';

    export let col: number;
    export let row: number;
    export let disabled_seats: string[];
    export let is_started_randomizing: boolean;
    let seats: number[][];
    let seats_table: HTMLElement;

    let loading = true;

    function getRandomInt(min: number, max: number) {
        min = Math.ceil(min);
        max = Math.floor(max);
        return Math.floor(Math.random() * (max - min + 1)) + min;
    }

    async function randomizeClassSeat() {
        // initialize the seats array
        seats = JSON.parse(JSON.stringify(Array(row).fill(Array(col).fill(0))));

        // mark the disabled seats
        disabled_seats.forEach((value) => {
            const [_x, _y] = value.split("|")
            const x = Number(_x);
            const y = Number(_y);
            try {
                seats[y][x] = -1;
            } catch {}
        });

        // randomize!
        let seats_pos: number[][] = [];
        seats.forEach((r, y) => r.forEach((value, x) => {
            if (value !== -1) {
                seats_pos.push([x, y]);
            }
        }));
        const number_of_students = col * row - disabled_seats.length;
        for (let class_num = 1; class_num <= number_of_students; class_num++) {
            const random_int = seats_pos.length - 1 <= 0 ? 0 : getRandomInt(1, seats_pos.length - 1);
            const random_seat = seats_pos[random_int];
            seats[random_seat[1]][random_seat[0]] = class_num;
            seats_pos.splice(random_int, 1);
        }
    }

    function goBack() {
        const back = confirm("저장하셨습니까? 돌아가면 결과는 날아갑니다.");
        if (back) {
            is_started_randomizing = false;
        }
    }

    function exportAsCSV() {
        // export as a CSV file
        let data = "";
        seats.forEach((r, y) => {
            r.forEach((value, x) => {
                data += value === -1 ? '""' : value;
                data += ",";
            });
            data = data.slice(0, -1)
            data += "\r\n";
        });
        data = data.slice(0, -1)
        let downloadLink = document.createElement("a");
		let blob = new Blob([data], { type: "text/csv;charset=utf-8" });
		let url = URL.createObjectURL(blob);
		downloadLink.href = url;
		downloadLink.download = `seats_${new Date().toLocaleDateString('ja-JP')}.csv`;
		document.body.appendChild(downloadLink);
		downloadLink.click();
		document.body.removeChild(downloadLink);
    }

    function exportAsPNG() {
        // export as a image file
        html2canvas(seats_table).then(canvas => {
            const link = document.createElement("a");
            link.href = canvas.toDataURL();
            link.download = `seats_${new Date().toLocaleDateString('ja-JP')}.csv`;
            link.click();
        });
    }

    onMount(async () => {
        // ask if to go out
        window.onbeforeunload = function() {
            return true;
        };

        await randomizeClassSeat();
        loading = false;
    });
</script>

<section>
{#if loading}
    <div>loading</div>
{:else}
    <h3>무작위 배치가 완료되었습니다!</h3>
    <table bind:this={seats_table}>
        {#each seats as r, y}
            <tr>
                {#each r as num, x}
                    {#if disabled_seats.includes(`${x}|${y}`)}
                        <td class="disabled" data-x={x} data-y={y}></td>
                    {:else}
                        <td data-x={x} data-y={y}><span>{num}</span></td>
                    {/if}
                {/each}
            </tr>
        {/each}
    </table>
    <div class="menu">
        <button on:click={goBack} class="back">돌아가기</button>
        <button on:click={exportAsCSV}>CSV로 저장</button>
        <button on:click={exportAsPNG}>PNG로 저장</button>
    </div>
{/if}
</section>

<style>
    section {
		display: flex;
		flex-direction: column;
		align-items: center;
        padding: 0 20px;
        padding-top: 50px;
	}

	table {
		margin-bottom: 50px;
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
        text-align: center;
        font-size: 20px;
	}

	.disabled {
		background-color: rgb(179, 179, 179);
	}

    .menu {
        display: flex;
        justify-content: center;
    }

    .menu button {
        margin: 0 5px;
        width: 100px;
        border: none;
        padding: 10px 0;
        border-radius: 5px;
        background-color: #62b87c;
        color: white;
        cursor: pointer;
    }

    .menu .back {
        background-color: gray;
    }
</style>