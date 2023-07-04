### page.svelte
import { enhance } from "$app/forms";
  const submitForm = ({ form, data, action, cancel, submitter }) => {
    let body = Object.fromEntries(data);
    submitter.type = "button";
    console.log("TTT")
    console.log(body)
    try {

    } catch (err) {
      submitter.type = "submit";
      cancel();
    }
    return async ({ result, update }) => {
      console.log(result);
      goto("/affairs_support/uniform/status");
    };
  };
<form method="POST" action="?/search" use:enhance={submitForm}>
  <input hidden bind:value={teamId} name="team_id" />
  <input hidden bind:value={processId} name="work_process_id" />
  </form>



  #### page.sever.js

export const actions = {
  search: async ({ request, locals }) => {
    const formData = await request.formData()
    let body = Object.fromEntries(formData)
    console.log(body)
    const test = await locals.sb
    .from(`uniform_orders`)
    .select()
    .is(`deleted_at`, null)
