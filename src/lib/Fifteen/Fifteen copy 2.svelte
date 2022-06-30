<script>
import {createEventDispatcher} from 'svelte';

  export let level = 3;

  const dispatch = createEventDispatcher();
  let cells, order, empty,  count, size;

  $: {
    count = level ** 2;
    empty = count -1;
    size = 100 / level;
    // order = Array.from({length: level}, (_, r) => Array.from({length: level}, (_, c) => level * r + c));
    order = Array.from({length: level}, (_, i) => i);
    cells = Array.from({ length: count }, () => ({}));
    bindOrder();
  }

  export const shuffle = () => {
    order.sort(() => 0.5 - Math.random());

    // normalize shuffle
    // if (!isSolvable()) {
    //   let i1 = order.indexOf(count - 1);
    //   let i2 = order.indexOf(count - 2);
    //   [order[i1], order[i2]] = [order[i2], order[i1]];
    // }

    bindOrder();
    cells = cells;
  };

  function isSolvable() {
    // проверка на нерешаемость пятнашек Лойда
    let sum = order.reduce((sum, cur, i, order) => {
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

  function bindOrder() {
    let inOrder;
    order.forEach((n, i) => {
      const is = n === i;
      const cell = cells[i];

      cell.inPlace = is;
      cell.row = Math.floor(n / level);
      cell.col = n % level;
      cell.num = n;

      inOrder = inOrder && is;
    })

    if (inOrder) 
      dispatch('order');
  }

  function cellClicked() {
    const cur =  order[this.dataset.index];
    const emp = order[count-1];
    // const cur = { i: this.dataset.index, p: order[this.dataset.index]};
    // const emp = { i: count - 1, p: order[count-1]};

    const diff = emp - cur;
    const factor = diff > 0 ? 1 : -1;
    const max = Math.max(cur, emp);
    const min = Math.min(cur, emp);

    //   // In one column
      if (cur%level === emp%level) {
        console.log('col');
        order[emp] = order[cur];
        order[cur] += level*factor;
        empty = cur;
        for (let i = min + level * factor; i < max; i += level) {
          console.log(i + ': ' + order[i] + ': ' + (order[i] + level*factor))
          order[i] += level*factor;

        }
      }
    //   // In one row
    //   else if (Math.abs(diff) < level)
    //     for (let i = cur + factor; i !== emp; i += factor)  
    //       order[i] += level*factor;
    //   else
    //     return;
    
    // order[cur] = order[emp];
    // order[emp] += factor;

    console.log(order);

    bindOrder();
    cells = cells;
  }
</script>

<div class="batlefield">
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
    /* background-color: #eee; */
    width: 100%;
    aspect-ratio: 1 / 1;
    max-width: 600px;
    margin: auto;
  }

  .cell {
    aspect-ratio: 1 / 1;
    position: absolute;
    transition: 0.2s;
  }
</style>
