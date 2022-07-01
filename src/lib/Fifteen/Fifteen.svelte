<script>
import {createEventDispatcher} from 'svelte';

  export let level = 3;

  const dispatch = createEventDispatcher();

  let cells, order, count, size;

  $: {
    count = level ** 2;
    size = 100 / level;
    order = Array.from({length: count}, (e, i) => i + 1);
    cells = Array.from({ length: count }, (e, i) => ({
      inPlace: true,
      row: Math.floor(i / level),
      col: i % level,
    }));
  }

  export const shuffle = () => {
    order.sort(() => 0.5 - Math.random());

    // normalize shuffle
    if (!isSolvable()) {
      let i1 = order.indexOf(count - 1);
      let i2 = order.indexOf(count - 2);
      [order[i1], order[i2]] = [order[i2], order[i1]];
    }

    order.forEach((n, i) => {
      const cell = cells[n - 1];
      cell.row = Math.floor(i / level);
      cell.col = i % level;
    });

    checkOrder();

    cells = cells;
  };

  function isSolvable() {
    // проверка на нерешаемость пятнашек Лойда
    let sum = order.reduce((sum, cur, i, arr) => {
      // если пустая ячейка и если уровень чётный,
      // вычисляем номер её ряда и прибавляем к счетчику
      if (cur === count && level % 2 === 0) {
        let row = Math.ceil((i + 1) / level);
        sum += row;
        return sum;
      }

      for (let n = i + 1; n < count; n++) {
        if (order[n] !== count && cur > order[n]) sum++;
      }
      return sum;
    }, 0);

    return sum % 2 === 0;
  }

  function checkOrder() {
    let inOrder = true;
    cells.forEach((cell, i) => {
      let is = (cell.row * level + cell.col) === i;
      cell.inPlace = is;
      inOrder = inOrder && is;
    });

    if (inOrder) 
      dispatch('order');
  }

  function cellClicked() {
    const i = Number(this.dataset.index);
    const current = cells[i];
    const empty = cells[count - 1];

    if (current.row !== empty.row && current.col !== empty.col)
      return;

    const prop = current.row === empty.row ? "col" : "row";
    const diff = empty[prop] - current[prop];

    if (Math.abs(diff) === 1) {
      [empty[prop], current[prop]] = [current[prop], empty[prop]];
      
    } else {
      const prop2 = prop === "row" ? "col" : "row";
      const factor = empty[prop] > current[prop] ? 1 : -1;
      const min = Math.min(empty[prop], current[prop]);
      const max = Math.max(empty[prop], current[prop]);

      cells.forEach((cell) => {
        if (
          cell[prop2] !== current[prop2] ||
          cell[prop] <= min ||
          cell[prop] >= max
        )
          return;
        cell[prop] += factor;
      });

      empty[prop] = current[prop];
      current[prop] += factor;
    }

    checkOrder();

    cells = cells;
  }
</script>

<div class="batlefield {$$props.class}" style="{$$props.style}">
  {#each cells as cell, index (cell)}
    {#if count !== index + 1}
      <div
        class="cell"
        style:width="{size}%"
        style:height="{size}%"
        style:top="{cell.row * size}%"
        style:left="{cell.col * size}%"
        data-index={index}
        on:click={cellClicked}
      >
        <slot {...cell} {index} />
      </div>
    {/if}
  {/each}
</div>

<style>
  .batlefield {
    position: relative;
    padding-bottom: 100%;
    margin: auto;
  }
  
  .cell {
    position: absolute;
    transition: 0.2s;
  }
</style>
