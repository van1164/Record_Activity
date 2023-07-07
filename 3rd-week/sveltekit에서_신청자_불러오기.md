<script>
  import _ from "lodash";
  import Title from "$lib/components/Title.svelte";
  import TeamDropdown from "$lib/components/TeamDropdown.svelte";
  import WorkProcessDropdown from "$lib/components/WorkProcessDropdown.svelte";
  import ConfirmModal from "$lib/components/ConfirmModal.svelte";
  import { onMount } from "svelte";
  import { DateTime } from "luxon";
  import { goto } from "$app/navigation";
  import ScrollableTableWrapper from "$lib/components/ScrollableTableWrapper.svelte";
  import { z } from "zod";
  import { enhance } from "$app/forms";
  import FindPersonDialog from "$lib/components/FindPersonDialog.svelte";
  import { ApprovalStatus } from "$lib/constants/ApprovalStatus.js";
  import { UniformCategory } from "$lib/constants/UniformCategory.js";
  import { UniformType } from "$lib/constants/UniformType.js";
  import { UniformSize } from "$lib/constants/UniformSize.js";
  export let data;

  const { user, workGroups, deliveryZones, startDate, finishDate } = data;
  const category = Object.keys(UniformCategory);
  const uniformType = Object.keys(UniformType);
  const uniformSize = Object.keys(UniformSize);
  let routes = [
    { name: "메인", href: "/" },
    { name: "총무지원", href: "/affairs_support" },
    { name: "작업복", href: "/affairs_support/uniform" },
    { name: "작업복 추가신청", href: "/affairs_support/uniform/order" },
  ];

  let teamId = user.team_id;
  let processId = user.work_process_id;
  let team = workGroups.find((workGroup) => workGroup.id === teamId);

  let selected;
  let selected_ind = 0;
  let answer = "";
  let selected_uniform = ["T"]
  let current_index =0
  let rows = [
    {
      name: null,
      amount: 0,
      reason: null,
      reason_detail: null,
      uniform_type: UniformType[UniformType[0]],
      category: UniformCategory[UniformCategory[0]],
      size: "-",
    },
  ];

  const onFoundPerson = (selectedUsers) => {
    if (!_.isEmpty(selectedUsers)) {
      const user = selectedUsers[0];
      console.log(user.e_number);
      rows[selected_ind].name = user.name;
    }
  };

  function addRow() {
    selected_uniform = [...selected_uniform,"T"]
    rows = [
      ...rows,
      {
        name: null,
        amount: 0,
        reason: null,
        uniform_type: UniformType[UniformType[0]],
        category: UniformCategory[UniformCategory[0]],
        size: "-",
      },
    ];
  }

  function delRow() {
    let len = rows.length;
    rows = rows.slice(0, len - 1);
    len = selected_uniform.length;
    selected_uniform = selected_uniform.slice(0, len - 1);
  }

  function delArr() {
    selected_uniform=[]
    rows = [
      {
        name: null,
        amount: 0,
        reason: null,
      },
    ];
  }
  const onChangedUniform= () =>{
    selected_uniform[current_index]=rows[current_index].uniform_type[0]
  }
  const submitForm = ({ form, data, action, cancel, submitter }) => {
    let body = Object.fromEntries(data);
    console.log(body)
    submitter.type = "button";
    try {
      if (body.users !== body.count) {
        formValidationReason.parse(body);
      } else {
        formValidation.parse(body);
      }
    } catch (err) {
      submitter.type = "submit";
      cancel();
    }
    return async ({ result, update }) => {
      alert("정상적으로 신청되었습니다.");
      goto("/affairs_support/uniform/order");
    };
  };

  const tempSave = ()=>{
    localStorage.setItem('uniform_rows',JSON.stringify(rows))
    localStorage.setItem('uniform_selected_uniforms',JSON.stringify(selected_uniform))
    alert("임시저장되었습니다.");
  }
  const tempClear = ()=>{
    localStorage.clear('uniform_rows')
    localStorage.clear('uniform_selected_uniforms')
    delArr()
    alert("초기화되었습니다.");
    goto("/affairs_support/uniform/order");
  }
  

  onMount(async () => {
    console.log(JSON.parse(localStorage.getItem('uniform_rows')))
    console.log(JSON.parse(selected_uniform = localStorage.getItem('uniform_selected_uniforms')))
   if (localStorage.getItem('uniform_rows')) rows = JSON.parse(localStorage.getItem('uniform_rows'))
   if (localStorage.getItem('uniform_selected_uniforms')) JSON.parse(selected_uniform = localStorage.getItem('uniform_selected_uniforms'))
  });


  const formValidation = z.object({
    user_name: z.string().nonempty({ message: "수령인을 선택해주세요." }),
    count: z.coerce.number().positive({ message: "주문은 1개 이상부터 가능합니다." }),
  });
</script>

<Title {routes} />
<form class="flex flex-col w-full pb-16" method="POST" action="?/create" use:enhance={submitForm}>
  <!-- 신청버튼 -->
  <div class="flex mt-2 gap-3 justify-end">
    <button class="btn btn-primary">신청</button>
    <!-- </form> -->
    <button type="button" class="btn btn-primary" on:click={tempSave}>임시저장</button>
    <button type="button" class="btn btn-primary" on:click={tempClear}>초기화</button>
  </div>

  <p class="text-hyundai-bold text-headline2 inline-block mt-2">등록자정보</p>
  <div class="w-full flex gap-4 mt-5 mb-4 max-xl:flex-col">
    <div class="flex flex-col w-1/3 max-xl:w-full">
      <span>사번</span>
      <input class="h-12 input input-bordered text-caption1" value={user.e_number} disabled />
    </div>
    <div class="flex flex-col w-1/3 max-xl:w-full">
      <span>팀</span>
      <input class="h-12 input input-bordered text-caption1" value={user.teams.name} disabled />
    </div>
    <div class="flex flex-col w-1/3 max-xl:w-full">
      <span>직책</span>
      <input class="h-12 input input-bordered text-caption1" value={user.position_name} disabled />
    </div>
    <div class="flex flex-col w-1/3 max-xl:w-full">
      <span>이름</span>
      <input class="h-12 input input-bordered text-caption1" value={user.name} disabled />
    </div>
  </div>

  <!-- 티켓번호, 진행사항, 버튼들 -->
  <div class="flex w-full justify-between">
    <p class="text-hyundai-bold text-headline2 inline-block mt-4">신청내역</p>
    <div class="flex">
      <div class="flex w-full justify-end p-2">
        <button type="button" on:click={addRow} class="btn btn-secondary">행추가</button>
      </div>
      <div class="flex w-full justify-end p-2">
        <button type="button" on:click={delRow} class="btn btn-secondary">행삭제</button>
      </div>
    </div>
  </div>

  <div class="flex-grow flex-col w-full pb-30 mb-1">
    <!-- 표 나오는 세션 -->
    <ScrollableTableWrapper>
      <!-- 데이지UI사용 -->
      <table class="table table-compact border-spacing-10 mb-10 flex w-full">
        <!-- 표 인덱스고정 -->
        <thead class="sticky top-0">
          <tr class="text-center">
            <td>No.</td>
            <td>이름</td>
            <td />
            <td>품목</td>
            <td>품목2</td>
            <td>사이즈</td>
            <td class=" w-2">수량</td>
            <td>사유</td>
            <td>첨부</td>
          </tr>
        </thead>
        {#each rows as row, rowIndex}
          <thead>
            <tr class="text-center font-hyundai-light">
              <td>{rowIndex + 1}</td>
              <td
                ><input
                  type="text"
                  name="{rowIndex}_name"
                  bind:value={row.name}
                  class="h-13 w-20 p-0.5 input input-bordered max-w-xs text-caption1"
                  style="width: 7rem;"
                  placeholder="이름"
                  readonly
                />
              </td>

              <td>
                <div>
                  <label for="find_person" class="btn btn-square" on:click={() => (selected_ind = rowIndex)}>찾기</label
                  >
                  <FindPersonDialog id="find_person" onSelected={onFoundPerson} isOnly={true} {workGroups} />
                </div>
              </td>

              <td>
                <select
                  name="{rowIndex}_uniform_type"
                  bind:value={row.uniform_type}
                  on:change={
                  current_index = rowIndex,
                  onChangedUniform}
                  class="input input-bordered text-caption1 w-32 max-xl:flex-col"
                  style="width: 5rem;"
                >
                  {#each uniformType as item}
                    <option value={UniformType[item].key}>{UniformType[item].value}</option>
                  {/each}
                </select>
              </td>

              <td>
                <select
                  name="{rowIndex}_category"
                  bind:value={row.category}
                  class="input input-bordered text-caption1 w-32 max-xl:flex-col"
                  style="width: 5rem;"
                >
                  {#each category as item}
                    <option value={UniformCategory[item].key}>{UniformCategory[item].value}</option>
                  {/each}
                </select>
              </td>

              <td>
                <select
                  name="{rowIndex}_size"
                  bind:value={row.size}
                  class="input input-bordered text-caption1 w-32 max-xl:flex-col"
                  style="width: 5rem;"
                >

                  {#each uniformSize as item}
                  {#if selected_uniform[rowIndex] == item[0]}
                    <option value={UniformSize[item].key}>{UniformSize[item].value}</option>
                    {/if}
                  {/each}

                </select>
              </td>

              <td class="w-2">
                <input
                  type="text"
                  name="{rowIndex}_amount"
                  bind:value={row.amount}
                  class="h-12 input input-bordered text-caption1"
                  style="width:3rem"
                  placeholder="수량"
                />
              </td>
              <td>
                <input
                  type="text"
                  name="{rowIndex}_reason"
                  bind:value={row.reason}
                  class="h-12 input input-bordered text-caption1"
                  style="width:16rem"
                  placeholder="사유"
                />
              </td>
              <td class="w-4">-</td>
            </tr>
          </thead>
        {/each}
      </table>
    </ScrollableTableWrapper>
  </div>
</form>
