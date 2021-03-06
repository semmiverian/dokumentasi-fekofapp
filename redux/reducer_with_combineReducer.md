# refactor reducer

<img width="500" src="https://github.com/miftakribo/dokumentasi-fekofapp/blob/master/redux/Screen%20Shot%202018-03-08%20at%2010.45.54.png" />

menurut saya structur reducer diatas udah mulai bloated, dan perlu direfactor ke dalam bentuk tree agar tidak memusingkan developer. contoh:
```javascript
{
  profile: { /*...*/ },
  register: { /*...*/ },
  builder: {
    sem: { /*...*/ },
    drm: { /*...*/ },
    shopping:{
      datasource: { /*...*/ },
      datafeedMapping: { /*...*/ },
      /*...*/
    }
  }
}

// alternative

{
  profile: { /*...*/ },
  register: { /*...*/ },
  builder: {
    datasource: { /*...*/ }, 
    campaignMapping: { /*...*/ },
    adsBuilder:{ /*...*/ }
  }
}
```

Ini berguna jika reducer yang kita buat terlalu besar dan komplex sehingga butuh direfactor dengan cara:
1. membuat 'reusable function': *fetchHandler*, *createReducer*.
2. memecah reducer yang komplex menjadi beberapa sub-reducer yang sederhana menggunakan *combineReducer*.

berikut merupakan contoh kasus reducer sebelum direfactor: (mungkin kasus pada contoh di bawah belum perlu di refactor karena tidak besar dan tidak kompleks)

```javascript
import * as C from './constant';

const dummiesData = { 
  userlist: {
    loading: false,
    data: [
      {id: 1, name: 'All Visitor'},
      {id: 2, name: 'Customer'},
      {id: 3, name: 'Past Buyer'},
    ]
  },
  haveExistingPortfolio: false,
  existingPortfolio: [
    {value: 0, label: 'Fetching Existing Portfolio...'}
  ]
};

const drmCampaignSetting = (state = {
  userlist: {
    loading: false,
    data: [],
    error: null
  },
  savelist: {
    loading: false,
    response: {},
    error: null
  },
  form: null,
  ...dummiesData
}, action) => {
  switch (action.type) {
    case C.LIST_REQ:
      return {
        ...state,
        userlist: {
          loading: true,
          data: [],
          error: null
        }
      };
      break;
    case C.LIST_FUL:
      return {
        ...state,
        userlist: {
          loading: false,
          data: action.payload,
          error: null
        }
      };
      break;
    case C.LIST_REJ:
      return {
        ...state,
        userlist: {
          loading: false,
          data: [],
          error: action.payload
        }
      };
      break;
    case C.FORM_SAVE:
      return {
        ...state,
        form: action.form
      };
      break;
    case C.DRM_REFRESH_CAMPAIGN_SETTING:
      return {
        ...state,
        savelist: {
          loading: false,
          response: {},
          error: null
        },
        form: null
      }
    case 'FETCH_EXISTING_PORTFOLIO':
      return {
        ...state,
        existingPortfolio: action.data.map(item => ({value: item.id, label: item.name})),
        haveExistingPortfolio: true
      }
      case C.SAVE_LIST_REQ:
      	return {
      		...state,
      		savelist: {
      			loading: true,
      			response: {},
      			error: null
      		}
      	}
      	break;
      case C.SAVE_LIST_FUL:
      	return {
      		...state,
      		savelist: {
      			loading: false,
      			response: action.payload,
      			error: null
      		}
      	}
      	break;
      case C.SAVE_LIST_REJ:
      	return {
      		...state,
      		savelist: {
      			loading: false,
      			response: {},
      			error: action.payload
      		}
      	}
      	break;
    default:
      return state;
  }
};

export default drmCampaignSetting;
```


setelah menggunakan direfactor menggunakan *combineReducer* dan *fetchHandler*:
```javascript
import * as C from './constant';
import { combineReducers } from 'redux'

const fetchHandler = {
  request: (data) => ({
    loading: true,
    data: data,
    error: null
  }),
  fulfilled: (data) => ({
    loading: false,
    data: data,
    error: null
  }),
  rejected: (data) => ({
    loading: false,
    data: [],
    error: ex
  })
}

const initialUserList = {
  loading: false,
  data: [],
  error: null
}
const userList = (state = initialUserList, action) => {
  switch (action.type) {
    case C.LIST_REQ:
      return fetchHandler.request([])
      break;
    case C.LIST_FUL:
      return fetchHandler.fulfilled(action.payload)
      break;
    case C.LIST_REJ:
      return fetchHandler.rejected(action.payload)
      break;
    default: return state
  }
}

const initialSaveList = {
  loading: false,
  response: {},
  error: null
}
const saveList = (state = initialSaveList, action) => {
  switch (action.type) {
    case C.SAVE_LIST_REQ:
     return fetchHandler.request({})
     break;
    case C.SAVE_LIST_FUL:
     return fetchHandler.fulfilled(action.payload)
     break;
    case C.SAVE_LIST_REJ:
     return fetchHandler.rejected(action.payload)
     break;
    case C.DRM_REFRESH_CAMPAIGN_SETTING:
      return initialSaveList
    default: return state
  }
}

const formReducer = (state = null, action) => {
  switch (action.type) {
    case C.FORM_SAVE:
      return action.form
      break;
    case C.DRM_REFRESH_CAMPAIGN_SETTING:
      return null
      break;
    default: return state
  }
}

const portofolioBool = (state = false, action) => {
  switch (action.type) {
    case 'FETCH_EXISTING_PORTFOLIO':
      return true
      break;
    case C.DRM_REFRESH_CAMPAIGN_SETTING:
      return false
      break;
    default: return state
  }
}

const initialPortofolio = [
  {value: 0, label: 'Fetching Existing Portfolio...'}
]
const portofolio = (state = initialPortofolio, action) => {
  switch (action.type) {
    case 'FETCH_EXISTING_PORTFOLIO':
      return action.data.map(item => ({value: item.id, label: item.name}))
      break;
    case C.DRM_REFRESH_CAMPAIGN_SETTING:
      return initialPortofolio
    default: return state
  }
}

const drmCampaignSetting = combineReducers({
  userlist: userList,
  savelist: saveList,
  form: formReducer,
  haveExistingPortfolio: portofolioBool,
  existingPortfolio: portofolio
})

export default drmCampaignSetting;
```
keuntungan menbuat *fetchHandler* adalah agar setiap fetch yang dilakukan memiliki flow yang sama, dan disarankan data yang diteruskan dari action adalah original response data dari API, kalaupun mesti ada normalize /  mapping ke bentuk yang dibutuhkan sebaiknya dilakukan di component atau menggunakan library selector [reselect](https://github.com/reactjs/reselect).

Dikarenakan *combineReducer* bisa nested sehingga berguna untuk memecah reducer yang kompleks menjadi bentuk2 reducer kecil yang lebih sederhana tanpa merubah structur reducer sebelumnya.

```javascript
rootReducer = combineReducers({
  router, // redux-react-router reducer
    account: combineReducers({
      profile: combineReducers({
         info, // reducer function
         credentials // reducer function
      }),
      billing // reducer function
    }),
    // ... other combineReducers
  })
});
```

setelah mengunakan createReducer: [sumber](https://redux.js.org/recipes/structuring-reducers/refactoring-reducers-example)

```javascript
export function createReducer(initialState, handlers) {
  return function reducer(state = initialState, action) {
    if (handlers.hasOwnProperty(action.type)) {
      return handlers[action.type](state, action)
    } else {
      return state
    }
  }
}
```

```javascript
import * as C from './constant';
import { combineReducers } from 'redux'
import { createReducer } from 'helpers/redux'

const fetchHandler = {
  request: (data) => ({
    loading: true,
    data: data,
    error: null
  }),
  fulfilled: (data) => ({
    loading: false,
    data: data,
    error: null
  }),
  rejected: (data) => ({
    loading: false,
    data: [],
    error: ex
  })
}

const initialUserList = {
  loading: false,
  data: [],
  error: null
}
const userList = createReducer(initialUserList, {
  [C.LIST_REQ]: (state, action) => fetchHandler.request([]),
  [C.LIST_FUL]: (state, action) => fetchHandler.fulfilled(action.payload),
  [C.LIST_REJ]: (state, action) => fetchHandler.rejected(action.payload)
})

const initialSaveList = {
  loading: false,
  response: {},
  error: null
}
const saveList = createReducer(initialSaveList, {
  [C.SAVE_LIST_REQ]: (state, action) => fetchHandler.request({}),
  [C.SAVE_LIST_FUL]: (state, action) => fetchHandler.fulfilled(action.payload),
  [C.SAVE_LIST_REJ]: (state, action) => fetchHandler.rejected(action.payload)
})

const formReducer = createReducer(null, {
  [C.FORM_SAVE]: (state, action) => action.form,
  [C.DRM_REFRESH_CAMPAIGN_SETTING]: (state, action) => null
})

const portofolioBool = createReducer(false, {
  'FETCH_EXISTING_PORTFOLIO': (state, action) => true,
  [C.DRM_REFRESH_CAMPAIGN_SETTING]: (state, action) => false
})

const initialPortofolio = [
  {value: 0, label: 'Fetching Existing Portfolio...'}
]
const portofolio = createReducer(initialPortofolio, {
  'FETCH_EXISTING_PORTFOLIO': (state, action) => action.data.map(item => ({value: item.id, label: item.name})),
  [C.DRM_REFRESH_CAMPAIGN_SETTING]: (state, action) => initialPortofolio
})

const drmCampaignSetting = combineReducers({
  userlist: userList,
  savelist: saveList,
  form: formReducer,
  haveExistingPortfolio: portofolioBool,
  existingPortfolio: portofolio
})

export default drmCampaignSetting;
```
