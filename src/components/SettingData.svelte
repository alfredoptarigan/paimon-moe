<script>
  import { server, ar, wl, loading } from '../stores/server';
  import { accounts, getAccountPrefix } from '../stores/account';
  import { readSave, updateSave, fromRemote } from '../stores/saveManager';
  import { onMount } from 'svelte';

  let unsubscribeServer = null;
  let unsubscribeAr = null;
  let unsubscribeWl = null;
  let firstLoad = true;

  $: if ($fromRemote) {
    readLocalData();
  }

  onMount(() => {
    readLocalData();
  });

  function readLocalData() {
    loading.set(true);

    const prefix = getAccountPrefix();

    firstLoad = true;

    if (unsubscribeServer) unsubscribeServer();
    if (unsubscribeAr) unsubscribeAr();
    if (unsubscribeWl) unsubscribeWl();

    console.log('setting read local');
    const accountsSave = readSave('accounts');
    const serverSave = readSave(`${prefix}server`);
    const arSave = readSave(`${prefix}ar`);
    const wlSave = readSave(`${prefix}wl`);
    if (accountsSave !== null) {
      const accountList = accountsSave.split(',').map((e) => ({
        label: `Account ${e.substring(7)}`,
        value: e,
      }));
      accounts.set([{ label: 'Main', value: 'main' }, ...accountList]);
    }
    if (serverSave !== null) {
      const currentServer = serverSave;
      server.set(currentServer);
    }
    if (arSave !== null) {
      ar.set(Number(arSave));
    }
    if (wlSave !== null) {
      wl.set(Number(wlSave));
    }

    unsubscribeServer = server.subscribe((val) => {
      if (firstLoad) return;

      const prefix = getAccountPrefix();
      console.log('server changed', val);
      updateSave(`${prefix}server`, val);
    });
    unsubscribeWl = wl.subscribe((val) => {
      if (firstLoad) return;

      const prefix = getAccountPrefix();
      console.log('wl changed', val);
      updateSave(`${prefix}wl`, val);
    });
    unsubscribeAr = ar.subscribe((val) => {
      if (firstLoad) return;

      const prefix = getAccountPrefix();
      console.log('ar changed', val);
      updateSave(`${prefix}ar`, val);
    });

    firstLoad = false;
    loading.set(false);
  }
</script>
