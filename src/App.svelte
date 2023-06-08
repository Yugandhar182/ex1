<script>
  import { onMount } from 'svelte';
  import { createEventDispatcher } from 'svelte';

  let gridContainer;
  let agGrid;
  let gridApi;
  let gridOptions;

  const columnDefs = [
    { headerName: 'ID', field: 'id' },
    { headerName: 'First Name', field: 'firstName', editable: true, filter: 'agTextColumnFilter' },
    { headerName: 'Surname', field: 'surname', editable: true, filter: 'agTextColumnFilter' },
    { headerName: 'Email', field: 'email', editable: true, filter: 'agTextColumnFilter' },
    { headerName: 'Mobile', field: 'mobile', editable: true, filter: 'agTextColumnFilter' },
    
  ];

  let rowData = [];
  let searchQuery = '';
  let isModalOpen = false;
  let newCandidate = {
    firstName: '',
    surname: '',
    email: '',
    mobile: ''
   
  };

  const dispatch = createEventDispatcher();

  function addCandidate() {
    isModalOpen = true;
  }

  function saveCandidate() {
    rowData.push(newCandidate);
    gridOptions.api.setRowData(rowData);

    // Make a POST request to add the new candidate to the API
    fetch('https://api.recruitly.io/api/candidate?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(newCandidate)
    })
      .then((response) => {
        if (!response.ok) {
          throw new Error(`API request failed with status ${response.status}`);
        }
        return response.json();
      })
      .then((addedData) => {
        console.log('New candidate added on the API:', addedData);
      })
      .catch((error) => {
        console.error('Error adding new candidate on the API:', error);
      });

    closeCandidateModal();
  }

  function closeCandidateModal() {
    isModalOpen = false;
    newCandidate = {
      firstName: '',
      surname: '',
      email: '',
      mobile: ''
      
    };
    dispatch('close');
  }

  function search() {
    const filterOptions = [
      { field: 'firstName', filter: gridOptions.api.getFilterInstance('firstName') },
      { field: 'surname', filter: gridOptions.api.getFilterInstance('surname') },
      { field: 'email', filter: gridOptions.api.getFilterInstance('email') },
      { field: 'mobile', filter: gridOptions.api.getFilterInstance('mobile') }
    ];

    filterOptions.forEach(({ field, filter }) => {
      if (filter) {
        filter.setModel({
          type: 'contains',
          filter: searchQuery
        });
        gridOptions.api.onFilterChanged();
      }
    });
  }

  onMount(() => {
    import('ag-grid-community').then((module) => {
      agGrid = module;
      gridOptions = {
        columnDefs: columnDefs,
        rowData: rowData,
        defaultColDef: {
          sortable: true,
          unSortIcon: true,
          resizable: true,
          flex: 1,
          minWidth: 100,
         
          editable: true
        },
        enableBrowserTooltips: true,
        pagination: true,
        paginationPageSize: 10,
        suppressRowClickSelection: true,
        rowSelection: 'multiple',
      
        onGridReady: function (params) {
          gridApi = params.api;
          gridApi.sizeColumnsToFit();
        },
        onCellValueChanged: function (params) {
          const { data } = params;
          const isNewCandidate = !data.id;

          // Make a PUT request to update the data on the API
          fetch(`https://api.recruitly.io/api/candidate?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E`, {
            method: isNewCandidate ? 'POST' : 'PUT',
            headers: {
              'Content-Type': 'application/json'
            },
            body: JSON.stringify(data)
          })
            .then((response) => {
              if (!response.ok) {
                throw new Error(`API request failed with status ${response.status}`);
              }
              return response.json();
            })
            .then((updatedData) => {
              console.log('Data updated on the API:', updatedData);
            })
            .catch((error) => {
              console.error('Error updating data on the API:', error);
            });
        }
      };
      new agGrid.Grid(gridContainer, gridOptions);

      fetch('https://api.recruitly.io/api/candidate/?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E')
        .then((response) => response.json())
        .then((data) => {
          if (Array.isArray(data.data)) {
            rowData = data.data.map((candidate) => ({
              id: candidate.id,
              firstName: candidate.firstName,
              surname: candidate.surname,
              email: candidate.email,
              mobile: candidate.mobile,
            }));
            gridOptions.api.setRowData(rowData);
          } else {
            console.error('Invalid API response:', data);
          }
        })
        .catch((error) => {
          console.error('Error fetching data:', error);
        });
    });
  });
</script>

<svelte:head>
  <script src="https://cdn.jsdelivr.net/npm/ag-grid-community@latest/dist/ag-grid-community.min.noStyle.js"></script>
  <link
    rel="stylesheet"
    href="https://cdn.jsdelivr.net/npm/ag-grid-community@latest/dist/styles/ag-grid.css"
  />
  <link
    rel="stylesheet"
    href="https://cdn.jsdelivr.net/npm/ag-grid-community@latest/dist/styles/ag-theme-alpine.css"
  />

  <link
    rel="stylesheet"
    href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css"
  />
</svelte:head>

<div class="container d-flex justify-content-center align-items-center vh-100">
  <div class="d-flex flex-column">
    <div class="mb-3">
      <input type="text" class="form-control" placeholder="Search" bind:value={searchQuery} on:input={search} />
    </div>
    <div class="mb-3">
      <button class="btn btn-primary" on:click={addCandidate}>Add Candidate</button>
    </div>
    <div id="datagrid" class="ag-theme-alpine" style="height: 600px; width: 800px;" bind:this={gridContainer}></div>

    {#if isModalOpen}
    <div class="modal show" tabindex="-1" style="display: block;">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Add Candidate</h5>
            <button type="button" class="btn-close" aria-label="Close" on:click={closeCandidateModal}></button>
          </div>
          <div class="modal-body">
            <div class="mb-3">
              <label for="firstNameInput" class="form-label">First Name</label>
              <input type="text" class="form-control" id="firstNameInput" bind:value={newCandidate.firstName} />
            </div>
            <div class="mb-3">
              <label for="surnameInput" class="form-label">Surname</label>
              <input type="text" class="form-control" id="surnameInput" bind:value={newCandidate.surname} />
            </div>
            <div class="mb-3">
              <label for="emailInput" class="form-label">Email</label>
              <input type="email" class="form-control" id="emailInput" bind:value={newCandidate.email} />
            </div>
            <div class="mb-3">
              <label for="mobileInput" class="form-label">Mobile</label>
              <input type="tel" class="form-control" id="mobileInput" bind:value={newCandidate.mobile} />
            </div>
          
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-primary" on:click={saveCandidate}>Save</button>
            <button type="button" class="btn btn-secondary" on:click={closeCandidateModal}>Close</button>
          </div>
        </div>
      </div>
    </div>
    <div class="modal-backdrop show"></div>
    {/if}
  </div>
</div>

<style>
  #datagrid {
    --ag-header-foreground-color: blue;
  }
  :global(.ag-header-cell) {
    font-size: 16px;
  }
  :global(.ag-header-group-cell) {
    border-right: 1px solid lightgray;
  }
</style>
