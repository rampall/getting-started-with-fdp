<script lang="ts">
  import { FdpStorage } from "@fairdatasociety/fdp-storage";
  import { onMount } from "svelte";
  import { get } from "svelte/store";
  import {
    createAppPodCode,
    createTodoItemCode,
    createTodoItemsDirectoryCode,
    createWalletCode,
    getAccountBalancesCode,
    listTodoItemsCode,
    initCode,
    listPodsCode,
    updateTodoCode,
  } from "./lib/snippets";
  import { getAddressBalances } from "./lib/utils";
  import {
    fdp,
    steps,
    wallet,
    importedWallet,
    podList,
    pods,
    sharedPods,
    appPod,
    todoItems,
    todoText,
    newTodo,
    type Todo,
    part,
  } from "./lib/store";
  import RunButton from "./lib/components/RunButton.svelte";
  import { config, TODOS_NS, TODO_ITEMS_DIR } from "./lib/config";
  import Code from "./lib/components/Code.svelte";
  import { utils } from "ethers";
  import { saveAs } from "file-saver";
  import type { ContentItem } from "@fairdatasociety/fdp-storage/dist/content-items/content-item";

  const init = () => {
    $fdp = new FdpStorage(config.beeUrl, config.batchId);
    console.log({ fdp: $fdp });
    $steps[1] = true;
  };

  const importWallet = () => {
    let phrase = prompt("Enter mnemonic phrase:");
    if (phrase) {
      $importedWallet = true;
      $fdp.account.setAccountFromMnemonic(phrase.trim());
      console.log({ phrase }, $fdp.account);
      $wallet = $fdp.account.wallet as any;
      $steps[2] = true;
    }
  };

  const createWallet = () => {
    try {
      $wallet = $fdp.account.createWallet();
      $steps[2] = true;
    } catch (err) {
      console.error(err);
    }
    console.log({ wallet: $fdp.account.wallet });
  };

  const createAppPod = async () => {
    try {
      $appPod = await $fdp.personalStorage.create(TODOS_NS);
      console.log({ appPod: $appPod });
      $podList = await $fdp.personalStorage.list();
    } catch (err) {}
    $steps[4] = true;
  };

  const listPods = async () => {
    const balance = await $fdp.ens.provider.getBalance($wallet.address);
    console.log({ balance: utils.formatEther(balance) });
    $podList = await $fdp.personalStorage.list();
    $pods = $podList.getPods();
    $sharedPods = $podList.getSharedPods();
    console.log({ pods: $pods, sharedPods: $sharedPods });
    $steps[3] = true;
    if ($importedWallet) {
      $steps[4] = true;
      $steps[5] = true;
      $steps[6] = true;
    }
  };

  const saveMnemonic = () => {
    var blob = new Blob(
      [JSON.stringify({ mnemonic: $wallet.mnemonic.phrase })],
      { type: "text/plain;charset=utf-8" }
    );
    saveAs(blob, "mnemonic.json");
    $steps[2.1] = true;
  };
  const createTodoItemsDirectory = async () => {
    $fdp.directory.create(TODOS_NS, TODO_ITEMS_DIR).then(() => {
      console.log(`Directory "${TODO_ITEMS_DIR}" created in "${TODOS_NS}" pod`);
    });
    $steps[5] = true;
  };
  const createTodoItem = async () => {
    const id = new Date().getTime();
    const uploadedTodoMetadata = await $fdp.file.uploadData(
      TODOS_NS,
      `${TODO_ITEMS_DIR}/todo_${id}.json`,
      JSON.stringify({ id, text: $todoText, done: false })
    );
    console.log({ uploadedTodoMetadata });
    $newTodo = uploadedTodoMetadata;
    $steps[6] = true;
    if ($steps[7]) {
      listTodoItems();
    }
  };
  const deleteTodoItem = async (todo: Todo) => {
    console.log({ todo });
    return $fdp.file
      .delete(TODOS_NS, `${TODO_ITEMS_DIR}/todo_${todo.id}.json`)
      .then(async () => {
        return listTodoItems();
      });
  };
  const toggleTodoItem = async (todo: Todo) => {
    console.log({ todo });
    $fdp.file
      .delete(TODOS_NS, `${TODO_ITEMS_DIR}/todo_${todo.id}.json`)
      .then(async () => {
        console.log("deleted ", { todo });
        try {
          const uploadedTodoMetadata = await $fdp.file.uploadData(
            TODOS_NS,
            `${TODO_ITEMS_DIR}/todo_${todo.id}.json`,
            JSON.stringify({ id: todo.id, text: todo.text, done: !todo.done })
          );
        } catch (err) {
          console.error(err);
        }
      });
  };
  const listTodoItems = async () => {
    const dir = await $fdp.directory.read(TODOS_NS, TODO_ITEMS_DIR);
    let promises = dir.content.map(async (file: ContentItem) => {
      return await get(fdp).file.downloadData(
        TODOS_NS,
        `${TODO_ITEMS_DIR}/${file.name}`
      );
    });
    $todoItems = (await Promise.all(promises)).map(
      (data) => data.json() as Todo
    );
    console.log({ content: dir.content, dir, todoItems: $todoItems });
    $steps[7] = true;
  };

  onMount(() => {
    console.log({ fdp: $fdp });
  });
</script>

<header>
  <h1>Getting started with FDP</h1>
  <nav>
    {#each ["Introduction", "Part 1: Building a Todo App", "Part 2: Sharing & Interoperability", "Part 3: Beyond fdp-play"] as nav, i}
      <a
        on:click={() => {
          $part = i;
        }}
        class:active={$part == i}>{nav}</a
      >
    {/each}
  </nav>
</header>
<main>
  {#if $part == 0}
    <h2>Overview</h2>
    <dl>
      <dt><a href="#why">Why FAIR</a></dt>
      <dd>
        provides an introduction to Fair Data Principles - the philosophy behind
        FDP.
      </dd>
      <dt><a href="#intro">Introducing FDP</a></dt>
      <dd>
        provides a quick overview of Fair Data Protocol and the various tools
        and libraries for developers available in its toolkit.
      </dd>
      <dt><a href="#about">About this guide</a></dt>
      <dd>
        <dl>
          <dt>
            <a
              href="#"
              on:click={() => {
                $part = 1;
              }}>Part 1</a
            >
          </dt>
          <dt>Building a Todo App</dt>
          <dd>
            <div class="notice">
              An interactive guide to build a simple Todo app covering various
              aspects of FDP storage & FDP Play such as
              <ul>
                <li>spinning a development environment with fdp-play</li>
                <li>creating / importing an FDP wallet account</li>
                <li>creating Pods</li>
                <li>creating directories within a Pod</li>
                <li>creating / deleting files within a Directory</li>
                <li>listing Pod contents</li>
              </ul>
            </div>
          </dd>
          <dt>
            <a
              href="#"
              on:click={() => {
                $part = 2;
              }}>Part 2</a
            >
          </dt>
          <dt>Sharing & Interoperability</dt>
          <dd>
            <div class="notice">
              Part 2 of the interactive guide covering additional aspects of FDP
              developement such as
              <ul>
                <li>top up FDP accounts</li>
                <li>registering FDP accounts on-chain to make them portable</li>
                <li>sharing pods / files / directories</li>
                <li>showcase interoperability between two example apps</li>
              </ul>
            </div>
          </dd>
          <dt>
            <a
              href="#"
              on:click={() => {
                $part = 3;
              }}>Part 3</a
            >
          </dt>
          <dt>
            Beyond <code class="in">fdp-play</code>
          </dt>
          <dd>
            <div class="notice">
              Part 3 of the guide covering additional aspects of FDP
              developement such as
              <ul>
                <li>moving from fdp-play to mainnet</li>
                <li>environment variables and configuration</li>
                <li>additonal resources and documentation</li>
                <li>github repositories</li>
              </ul>
            </div>
          </dd>
        </dl>
      </dd>
    </dl>
    <h2 id="why">Why FAIR</h2>
    <p>TODO</p>
    <h2 id="intro">Introducing FDP</h2>
    <dl>
      <dt>Fair Data Protocol</dt>
      <dd>
        Developed under Fair Data Society, Fair Data Protocol aims to provide
        web3 storage for dApps to connect and use, allowing for fair access and
        re-use of data on a global scale.
      </dd>
      <dt>FDP Storage</dt>
      <dd>
        FDP Storage is a serverless web3 filesystem for organizing users'
        personal data implemented in Typescript.
      </dd>
      <dt>FDP Play</dt>
      <dd>
        FDP Play is a CLI tool to spin up local development Bee cluster and FDP
        environment with Docker
      </dd>
      <dt>...</dt>
      <dd>TODO</dd>
    </dl>
  {:else if $part == 1}
    <section>
      <h2>Building a Todo App</h2>
      <h3>Prerequisites</h3>
      <div class="notice">
        1. Make sure <code>fdp-play</code> is installed and running:
        <p><code>{"$ fdp-play start --fairos"}</code></p>
        2. Open developer console to follow logs.
      </div>
    </section>
    <section>
      <h2>1. Create a FdpStorage instance:</h2>
      <Code source={initCode} />
      <RunButton
        step="1"
        handler={init}
        actionText="Create an fdpStorage instance"
      />
      {#if $fdp}
        <h3>1.1 List accounts and balances:</h3>
        <Code source={getAccountBalancesCode} />
        <RunButton
          step="1.1"
          handler={() => {
            $steps[1.1] = true;
          }}
          actionText="Show Blockchain (Ganache) Addresses and Balances"
        />
        {#if $steps[1.1]}
          <p class="notice">
            <strong>Ganache blockchain - Accounts & Balances :</strong>
            {#await getAddressBalances($fdp)}
              <p>...waiting</p>
            {:then accounts}
              <table>
                <tr><th>#</th><th>Address</th><th>Balance in ETH</th></tr>
                {#each accounts as account, i}
                  <tr
                    ><td>{i + 1}</td><td>{account.address}</td><td
                      ><b>{account.balance} eth</b></td
                    ></tr
                  >
                {/each}
              </table>
            {/await}
          </p>
        {/if}
      {/if}
    </section>
    {#if $steps[1.1]}
      <section>
        <h2>2. Create an FDP wallet:</h2>
        <Code source={createWalletCode} />
        <RunButton
          step="2"
          handler={createWallet}
          actionText="Create an FDP wallet"
        />
        or <RunButton
          step="2.1"
          handler={importWallet}
          actionText="Import an FDP wallet"
        />
        {#if $steps[2]}
          <dl class="notice">
            <dt><code class="in">wallet.address</code>:</dt>
            <dd><strong>{$wallet.address}</strong></dd>
            <dt><code class="in">wallet.mnemonic.phrase</code>:</dt>
            <dd>
              <em>{$wallet.mnemonic?.phrase || "Empty (imported wallet)"}</em>
            </dd>
          </dl>
          {#if !$importedWallet}
            <h3>2.1 Save mnemonic phrase:</h3>
            <RunButton
              step="2.1"
              handler={saveMnemonic}
              actionText="Save mnemonic phrase"
            />
          {/if}
        {/if}
      </section>
    {/if}
    {#if $importedWallet || ($steps[2] && $steps[2.1])}
      <h2>3. List existing pods:</h2>
      <Code source={listPodsCode} />
      <RunButton step="3" handler={listPods} actionText="List existing pods" />
    {/if}
    {#if $steps[3]}
      {#if $pods}
        <dl class="notice">
          <dt><code>Pods</code>:</dt>
          <dd>
            <ol>
              {#each $podList.getPods() as pod}<li>
                  <strong>{pod.name}</strong>
                </li>{:else}
                None
              {/each}
            </ol>
          </dd>
          <dt><code>Shared pods</code>:</dt>
          <dd>
            <ol>
              {#each $podList.getSharedPods() as pod}<li>
                  <strong>{pod.name}</strong>
                </li>{:else}
                None
              {/each}
            </ol>
          </dd>
        </dl>
      {/if}
      <section>
        <h2>4. Create app pod <code class="in">{TODOS_NS}</code>:</h2>
        This<code class="in">{TODOS_NS}</code> pod will serve as the namespace
        for our dapp.
        <Code source={createAppPodCode} />
        <RunButton
          step="4"
          handler={createAppPod}
          actionText="Create an App Pod"
        />
        {#if $appPod}
          <p class="notice">
            The <strong>{$appPod.name}</strong> pod has been created!
          </p>
        {/if}
      </section>
      {#if $steps[4]}
        <section>
          <h2>
            5. Create a directory <code class="in">{TODO_ITEMS_DIR}</code>:
          </h2>
          This<code class="in">{TODO_ITEMS_DIR}</code> directory inside the
          <code class="in">{TODOS_NS}</code>
          pod will hold each of our todo items as individual files.
          <Code source={createTodoItemsDirectoryCode} />
          <RunButton
            step="5"
            handler={createTodoItemsDirectory}
            actionText="Create a '{TODO_ITEMS_DIR}' directory"
          />
          {#if $steps[5]}
            <p class="notice">
              The <code class="in">{TODO_ITEMS_DIR}</code> directory has been
              created in <code class="in">{TODOS_NS}</code> pod!
            </p>
          {/if}
        </section>
      {/if}
      {#if $steps[5]}
        <section>
          <h2>6. Create a todo item (as a file):</h2>
          <Code source={createTodoItemCode} />
          <RunButton
            step="6"
            handler={createTodoItem}
            actionText="Create a Todo Item"
          />
          {#if $steps[6]}
            <RunButton
              step="6.1"
              handler={createTodoItem}
              actionText="Create another Todo Item"
            />
            {#if $newTodo}
              <dl class="notice">
                <em>New Todo has been added!</em>
                <dt><code>filePath</code>:</dt>
                <dd>{$newTodo.filePath}</dd>
                <dt><code>filename</code>:</dt>
                <dd>{$newTodo.fileName}</dd>
              </dl>
            {/if}
          {/if}
        </section>
      {/if}
      {#if $steps[6]}
        <section>
          <h2>7. List all todo items:</h2>
          <Code source={listTodoItemsCode} />
          <RunButton
            step="7"
            handler={listTodoItems}
            actionText="List all Todo Items"
          />
          {#if $steps[7] && $todoItems.length}
            <table class="notice">
              <tr><td>#</td><th>id</th><th>todo</th><th>done</th></tr>
              {#each $todoItems as todo, i}
                <tr
                  ><td>{i + 1}</td><td>{todo.id}</td><td>{todo.text}</td><td
                    >{todo.done}</td
                  ></tr
                >
              {/each}
            </table>
          {/if}
        </section>
      {/if}
      {#if $steps[7]}
        <section>
          <h2>8. Deleting a todo:</h2>
          <Code source={updateTodoCode} />
          {#if $steps[7] && $todoItems.length}
            <table class="notice">
              <tr
                ><th>#</th><th>id</th><th>todo</th><th>done?</th><th>actions</th
                ></tr
              >
              {#each $todoItems as todo, i}
                <tr>
                  <td>{i + 1}</td>
                  <td>{todo.id}</td>
                  <td>{todo.text}</td>
                  <td>
                    <input
                      type="checkbox"
                      bind:checked={todo.done}
                      on:change={(e) => {
                        console.log({ e, todo });
                        toggleTodoItem(todo);
                      }}
                    />
                  </td>
                  <td>
                    <button
                      on:click={() => {
                        deleteTodoItem(todo);
                      }}
                    >
                      delete
                    </button>
                  </td>
                </tr>
              {/each}
            </table>
          {/if}
        </section>
      {/if}
    {/if}
  {:else if $part == 2}
    <h2>Sharing & Interoperability</h2>
    <div class="notice">
      Part 2 of the interactive guide covering additional aspects of FDP
      developement such as
      <ul>
        <li>top up FDP accounts</li>
        <li>registering FDP accounts on-chain to make them portable</li>
        <li>sharing pods / files / directories</li>
        <li>showcase interoperability between two example apps</li>
      </ul>
    </div>
    <h2>TODO</h2>
  {:else if $part == 3}
    <h2>Beyond <code class="in">fdp-play</code></h2>
    <div class="notice">
      Part 3 of the guide covering additional aspects of FDP developement such
      as
      <ul>
        <li>moving from fdp-play to mainnet</li>
        <li>environment variables and configuration</li>
        <li>additonal resources and documentation</li>
        <li>github repositories</li>
      </ul>
    </div>
    <h2>TODO</h2>
  {/if}
</main>
<div id="footer" />
<style>
</style>
