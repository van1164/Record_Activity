  import { onMount,afterUpdate } from "svelte";
    import { page } from "$app/stores";
      let nav
  const myurl = $page.url.href.replace(/^(?:\/\/|[^/]+)*/, "")
  




  onMount(async() => {
    nav = localStorage.getItem(myurl)
     nav = JSON.parse(nav)
     console.log(nav)
  if(nav)  { 
    selectedTeamId = nav.selectedTeamId
    selectedProcessId = nav.selectedProcessId
}
  else{  
    selectedTeamId = user.teams.id;
    selectedProcessId = user.work_process.id;}
  if (location.hash) {
      const hash = location.hash.replace("#id_", "").split("_");
      selectedTeamId = hash[0];
      selectedProcessId = hash[1];
    }
    await search();
  });


  afterUpdate(async() => {
    let send_data={'selectedTeamId':selectedTeamId,'selectedProcessId':selectedProcessId}
    await localStorage.setItem(myurl,JSON.stringify(send_data))
    nav = await localStorage.getItem(myurl)
  });
