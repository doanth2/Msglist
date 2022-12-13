# Msglist
-----
index.tsx
------
import React, { useRef, useState } from "react";
import Head from 'next/head';
import * as Layout from "../../../components/Layout/index";
import { useDispatch } from "react-redux";
import Grid from '@mui/material/Grid';
import Button from "@mui/material/Button";
import Box from '@mui/material/Box';
import * as store from '../../../stores/Store';

import { format } from 'date-fns' 
import ja from 'date-fns/locale/ja'

import * as commonTips from "../../../constants/CommonTips";
import { StyledHeader, StyledTitleHeader } from "../../../styles/Styled";

import TipsIcon from "../../../components/Common/Tips";
import DataGrid_list from "../../../components/MsgList/Datagrid";
import { InputSelect } from "../../../components/Atoms/Select/Select";
import { useCallClassList } from "../../../features/CallClass/Selector";
import { useInputMsgConditions, useMsgList } from "../../../features/MsgList/Selector";
import { fetchAsyncMsgList } from "../../../features/MsgList/Operation";
import { fetchAsyncCallClassList } from "../../../features/CallClass/Operation";
import msgListSlice from "../../../features/MsgList/Slice";
import { TextField } from "@mui/material";
import { styled } from "@mui/styles";

type OnChangeEvent = React.ChangeEvent<HTMLInputElement>;

export const Main= ():JSX.Element  => {

  // useDispatch で store に紐付いた dispatch が取得できます
  const dispatch: store.AppDispatch = useDispatch();
  const conditions = useInputMsgConditions();
  const callClassList = useCallClassList();
  const msgList = useMsgList();
  const [dispDay, setDispDay] = useState(format(conditions.callDate, 'yyyy/MM/dd'));           // 移動日

  // [スタブ]部署コードが業務の場合はボタン、コンボボックスを非表示
  const busho = "00062";
//  const busho = "00061";

  // useEffectとは、関数の実行タイミングをReactのレンダリング後まで遅らせるhook
  React.useEffect(() => {
    /// 第1引数には実行させたい副作用関数を記述
    /// 第2引数には副作用関数の実行タイミングを制御する依存データを記述
    const promise = async() => {
      await dispatch(fetchAsyncCallClassList(conditions.title));
      await dispatch(fetchAsyncMsgList(conditions));
    };      
    promise();
  },[dispatch])

  // ************************
  // 移動日バリデーション用
  // ************************
  const dispDayValidPattern = "^[0-9/]+$";
  const dispDayRef = useRef<HTMLInputElement>(null);
  const [dispDayError, setDispDayError] = useState(false);

  // ************************
  // バリデーションチェック
  // ************************
  const formValidation = () : boolean => {
    let valid = true;

    // 移動日
    const valDay = dispDayRef?.current;
    if(valDay) {
      const ok = valDay.validity.valid;
      setDispDayError(!ok);
      valid &&= ok;
    }

    return valid;
  };

  // 状態によってボタンテキストを変更
  const completeButtonText = conditions.isShowComplete? "完了分を出さない" : "完了分を出す";
  const autoButtonText = conditions.isShowAuto? "本人送信分を出す" : "自動送信分を出す";
  
  // コール区分コンボボックス
  const handleOnChangeCallClass = (key:string,column:string,value:string | number) => {
    const newConditions = {...conditions,callClass:value.toString()};
    dispatch(msgListSlice.actions.conditionsChanged(newConditions));
  };

  // 本日・最新ボタン
  const handleToday = (newDate:Date) => {
    const newConditions = {...conditions,isToday:true,callDate:newDate,isMove:false};
    dispatch(msgListSlice.actions.conditionsChanged(newConditions)); 

    // 移動日テキストに本日日付
    setDispDay(format(new Date(), 'yyyy/MM/dd'));

    // リクエスト処理
    const promise = async() => {
      await dispatch(fetchAsyncMsgList(newConditions));
    }
    promise();
  };
  
  // 完了分を出すボタン
  const handleIsShowComplete = () => {
    dispatch(msgListSlice.actions.conditionsChanged({...conditions,isShowComplete: !(conditions.isShowComplete)})); 
  };

  // 自動送信分を出すボタン
  const handleIsShowAuto = (value:boolean) => {
    const newConditions = {...conditions,isShowAuto: !(conditions.isShowAuto)};
    dispatch(msgListSlice.actions.conditionsChanged({...conditions,isShowAuto: !(conditions.isShowAuto)})); 

    // リクエスト処理
    const promise = async() => {
      await dispatch(fetchAsyncMsgList(newConditions));
    };
    promise();
  };
  
  // 移動ボタン
  const handleDispDay= (newDate:string) => {
    if (formValidation()){
      var isToday
      var isShow = conditions.isShowComplete
      var isMove = true;
      if(newDate == format(new Date(), 'yyyy/MM/dd'))
      {
        isToday = true;
      } else
      {
        isToday = false;
        isShow = true;
      }

      var valDay = new Date(newDate);

      const newConditions = {...conditions,callDate:valDay, isToday: isToday, isShowComplete: isShow,isMove:isMove};
      dispatch(msgListSlice.actions.conditionsChanged(newConditions));

      // リクエスト処理
      const promise = async() => {
        await dispatch(fetchAsyncMsgList(newConditions));
      }
      promise();
    }
  }

  return(
    <React.Fragment>
      <Head>
        <title>伝言状況</title>
      </Head>
      <br></br>
      <br></br>
      <Grid container alignItems='right' justifyContent='right'>
        <TipsIcon
          title={commonTips.CST_TIPS_TITLE_MSGLIST}
          content={commonTips.CST_TIPS_CONTENT_MSGLIST}
        >
        </TipsIcon>
      </Grid>
      <Grid>
        <StyledHeader>
          <span>
            <strong>
              {format(conditions.callDate, 'M月d日(E)', {locale: ja})}
            </strong>
          </span>
          　
          {/* [スタブ]部署が業務の場合のみコンボボックス表示 */}
          {busho === "00062" &&
            <InputSelect 
              list={callClassList}
              title="区分："
              name="nm1"
              indx="callClassName"
              value="cd1"
              column="callClassName"
              unqkey="cd1"
              defaultValue={conditions.callClass}
              onChange={handleOnChangeCallClass}
              options={{blank:true,all:false}}
            />
          }
        </StyledHeader>
      </Grid>
      <br></br>
      <Box
        sx = {{
          bgcolor: 'info.main',
          color: 'background.paper',
          p: 2 ,
          textAlign: 'center',
          maxWidth : "100%",
        }}
      >
        <StyledTitleHeader>
          <strong>伝言一覧</strong>
        </StyledTitleHeader>
      </Box>
      <Grid>
        <Box
          sx = {{
            height: '90%',
            width: '100%',
          }}
        >
          <DataGrid_list 
            list = {msgList}
          />
        </Box>
      </Grid>
      <br></br>
      <Box
        sx = {{
          height: '100%',
          width: '100%',
        }}
      >
        <Grid container alignItems='center' justifyContent='center'>
          {/* [スタブ]部署が業務の場合のみコンボボックス表示 */}
          {busho === "00062" &&
            <Button
              variant="contained"
              color="primary"
              onClick={() => handleIsShowAuto(conditions.isShowAuto)}
              size='large'
            >
              {autoButtonText}
            </Button>
          }
          　
          <Button
            disabled={!conditions.isToday}
            variant="contained"
            color="primary"
            onClick={() => handleIsShowComplete()}
            size='large'
          >
            {completeButtonText}
          </Button>
          　
          <Button
            variant="contained"
            color="primary"
            onClick={() => handleToday(new Date())}
            size='large'
          >
            本日・最新
          </Button>
          　
            <TextField
              required
              id = "dispDay"
              label = "移動日（yyyy/mm/dd）"
              variant = "filled"
              size = "small"
              inputRef = { dispDayRef }
              inputProps = {{ minLength: 10, maxLength: 10, pattern: dispDayValidPattern }}
              value = {dispDay}
              onChange = {(e: OnChangeEvent) => setDispDay(e.target.value)}
              error = { dispDayError }
              helperText = { dispDayError && dispDayRef?.current?.validationMessage}
            />
          　
          <Button
            variant="contained"
            color="primary"
            onClick={() => handleDispDay(dispDay)}
            size='large'
          >
            移動
          </Button>
          <span>　未完了件数：{msgList.filter(x => x.resultFlg === false).length}件</span>
          {conditions.isMove === true &&
              <span>
                <strong style={{color:"red"}}>
                  　移動日で指定した日付のみ表示しています
                </strong>
              </span>
            }
        </Grid>
      </Box>
    </React.Fragment>
  )
};

export const Index = ():JSX.Element => {
  return (
      <Layout.Index mainComponent={<Main />} title='伝言入力' />
  );
};

export default Index;

----------------
layout common
\\\\---------
import  * as React from 'react';
import MenuList from '../../components/Menu/Atoms/MenuList';
import  { useState } from 'react';
import { styled, useTheme, Theme, CSSObject } from '@mui/material/styles';
import Box from '@mui/material/Box';
import MuiDrawer from '@mui/material/Drawer';
import CssBaseline from '@mui/material/CssBaseline';
import MuiAppbar, { AppBarProps as MuiAppBarProps } from '@mui/material/AppBar';
import Toolbar from '@mui/material/Toolbar';
import Divider from '@mui/material/Divider';
import IconButton from '@mui/material/IconButton';
import clsx from 'clsx';

// アイコン関連
import MenuIcon from '@mui/icons-material/Menu';
import ChevronLeftIcon from '@mui/icons-material/ChevronLeft';
import MenuItems from '../../components/Menu/CallCenter';

const backgroundColor = "##FFFFFF";

interface ComponentProps {
  mainComponent:JSX.Element,
  title:string,
};

const drawerWidth = 240;

// #region メニューのオープン／クローズ設定
const openedMixin = (theme: Theme): CSSObject => ({
    width: drawerWidth,
    transition: theme.transitions.create('width', {
        easing: theme.transitions.easing.sharp,
        duration: theme.transitions.duration.enteringScreen,
    }),
    overflowX: 'hidden',
});

const closedMixin = (theme: Theme): CSSObject => ({
    transition: theme.transitions.create('width', {
        easing: theme.transitions.easing.sharp,
        duration: theme.transitions.duration.leavingScreen,
    }),
    overflowX: 'hidden',
    width: `calc(${theme.spacing(7)} + 1px )`,
    [theme.breakpoints.up('sm')]: {
        width: `calc(${theme.spacing(8)} * 1px )`,
    },
});

interface AppBarProps extends MuiAppBarProps {
    open?: boolean;
};

const AppBar = styled(MuiAppbar, {
    shouldForwardProp: (prop) => prop !== 'open',
})<AppBarProps>(({ theme, open }) => ({
    zIndex: theme.zIndex.drawer + 1,
    transition: theme.transitions.create(['width', 'margin'], {
        easing: theme.transitions.easing.sharp,
        duration: theme.transitions.duration.leavingScreen,
    }),
    ...open && {
        marginLeft: drawerWidth,
        width: `calc(100% - ${drawerWidth}px)`,
        transition: theme.transitions.create(['width', 'margin'], {
            easing: theme.transitions.easing.sharp,
            duration: theme.transitions.duration.enteringScreen,
        }),
    },
}));

const DrawerHeader = styled('div')(({theme}) => ({
    display: 'flex',
    alignItems: 'center',
    justifyContent: 'flex-end',
    padding: theme.spacing(0,1),
    ...theme.mixins.toolbar,
}));

const Drawer = styled(MuiDrawer, { shouldForwardProp: (prop) => prop !== 'open'}) (
    ({ theme, open }) => ({
        width: drawerWidth,
        flexShrink: 0,
        whiteSpace: 'nowrap',
        boxSizing: 'border-box',
        ...(open && {
            ...openedMixin(theme),
            '& .MuiDrawer-paper': openedMixin(theme),
        }),
        ...(!open && {
            ...closedMixin(theme),
            '& .MuiDrawer-paper': closedMixin(theme),
        }),
    }),  
);
// #endregion


export interface MenuItem {
    id: number;
    url: string;
    icon: JSX.Element;
    text: string;
};

export interface MenuProps {
    items: MenuItem[]
};

const StyleForm = styled('form')(({theme}) => ({
    paddingTop:theme.spacing(0),
    backgroundColor:backgroundColor,
  }));
  
  const StyleContent = styled("div")({
    width:'100%',
    overflowX: 'hidden',
  });
  

export const Index = (mainComponentProps:ComponentProps) => {
    // サイドメニューの開閉状態
    const theme = useTheme();
    const [open, setOpen] = useState(true);
    const{ mainComponent,title } = mainComponentProps;

    // オープンクリック
    const handleDrawerOpen = () => {
        setOpen(true);
    };

    // クローズクリック
    const handleDrawerClose = () => {
        setOpen(false);
    };

    return (
      <React.Fragment>
            <Box
                sx = {{ display: 'flex' }}
            >
                <CssBaseline />
                <AppBar
                    position = 'fixed'
                    open={open}
                >
                    <Toolbar>
                        <IconButton
                            edge="start"
                            color='inherit'
                            onClick = {handleDrawerOpen}
                            sx = {{
                                mr: 2,
                                ...(open && { display: 'none'}),
                            }}
                        >
                            <MenuIcon/>
                        </IconButton>
                    </Toolbar>
                </AppBar>
                <Drawer
                    variant = "permanent"
                    open={open}
                    sx = {{
                        '& .MuiDrawer-paper' : {
                            backgroundColor: '#FFFFCC',
                            boxSizing: 'border-box',
                        },
                    }}
                >
                    <DrawerHeader>
                        <IconButton
                            onClick = {handleDrawerClose}
                            sx = {{
                                color: '#000000',
                            }}
                        >
                            {<ChevronLeftIcon/>}
                        </IconButton>
                    </DrawerHeader>
                    
                    {MenuItems().map((menu) => {
                        return (
                            <MenuList key={menu.id} url={menu.url} icon={menu.icon} text={menu.text} color='#87CEFA'/>
                        )
                    })}

                    <Divider />
                </Drawer>
                <Box component='main' sx ={{flexGrow: 1, p: 3}}>
                    <StyleForm>
                        <main className={clsx(StyleContent)}>
                            {mainComponent}
                        </main>
                    </StyleForm>
                </Box>
            </Box>
        </React.Fragment>
    );
};

export default Index;

-------------store
----
import { configureStore, combineReducers,ThunkAction,Action } from '@reduxjs/toolkit';
import logger from 'redux-logger';
import todoSlice from '../features/TodoList/Slice';
import callClassSlice from '../features/CallClass/Slice';
import operatorReducer from '../features/Operator/Slice';
import prefectureReducer from '../features/Prefecture/Slice';
import completeClassSlice from '../features/CompleteClass/Slice';
import todayReceptionSlice from '../features/TodayReception/Slice';
import msgListSlice from '../features/MsgList/Slice';
import msgKbnReducer from '../features/InputMsg/Slice';

const NODE_ENV = process.env.NODE_ENV;

const rootReducer = combineReducers({
  todo: todoSlice.reducer,
  callClass:callClassSlice.reducer,
  //operator:operatorSlice.reducer,
  operator:operatorReducer,
  prefecture:prefectureReducer,
  completeClass:completeClassSlice.reducer,
  todayReception:todayReceptionSlice.reducer,
  msgList:msgListSlice.reducer,
  msgKbn:msgKbnReducer,
})

export const store = configureStore({
    reducer: rootReducer,
    //middleware: (NODE_ENV === 'production')? [ ...getDefaultMiddleware({...getDefaultMiddleware,serializableCheck: false})] : [ ...getDefaultMiddleware({...getDefaultMiddleware,serializableCheck: false}), logger]
    middleware: (NODE_ENV === 'production')? 
                 (getDefaultMiddleware) => getDefaultMiddleware({
                  serializableCheck: false,
                 }).concat(logger)
              : (NODE_ENV === 'development')?
                 (getDefaultMiddleware) => getDefaultMiddleware({
                  serializableCheck: false,
                 }).concat(logger)
              : (getDefaultMiddleware) => getDefaultMiddleware({
                serializableCheck: false,
                }).concat(logger),
})

export type RootState = ReturnType<typeof rootReducer>;
export type AppDispatch = typeof store.dispatch;
export type AppThunk<ReturnType = void> = ThunkAction<
  ReturnType,
  RootState,
  unknown,
  Action<string>
>;
---------style page
\\\\\\\\\---------

import { styled } from '@mui/material/styles';

// ヘッダ部エリアのスタイル設定
//  （検索条件のエリア）
export const StyledHeader = styled("div")(({theme}) => ({
    padding:0,
    paddingLeft: theme.spacing(2),
    paddingTop: '10px',
    fontSize: 24,
  }));

// 一覧画面用ヘッダタイトルのスタイル設定
export const StyledTitleHeader = styled("div")({
  fontSize: 20,
});

// 入力項目のサンプル表記のスタイル設定
export const StyledExample = styled('text')({
  color:'blue',
});

export const StyledDataGrid = {
  grid: {
    '.MuiDataGrid-toolbarContainer': {
      borderBottom: 'solid 1px rgba(224, 224, 224, 1)'  ,

   
    },
     // 列ヘッダに背景色を指定
    '.MuiDataGrid-columnHeaders': {
      backgroundColor: '#40b4c8', 
      color: '#fff',
      fontSize: '1.2rem'
    },
    '.MuiDataGrid-row': {
      fontSize: '1.1rem'
    },
     //　ヘッダ選択時のフォーカス枠の切れを修正
    '.MuiDataGrid-columnHeader:focus-within': {
      outlineOffset: -3,
      outline: 'none'
    },
    //　データ選択時のフォーカス枠の切れを修正
    '.MuiDataGrid-cell:focus': {
      outline: 'none'
    },
    height:592,
    overflowX: 'auto',
  
  },
};
-------------------------------
tips -----common
\\\----
import InfoIcon from '@mui/icons-material/InfoRounded';
import { Tooltip, Typography } from '@mui/material';
import IconButton from "@mui/material/IconButton";
import {styled as iconstyled} from "@mui/styles";
import Modal from "@mui/material/Modal";
import React from 'react';
import Box from '@mui/system/Box';


// tips用アイコンスタイル設定
const IconStyle = iconstyled(InfoIcon)({
    width:35,
    height:35,
 });

// モーダルスタイル設定
const ModalStyle = {
    position: 'absolute',
    top: '10%',
    right: 100,
    width: 550,
    maxHeight:700,
    bgcolor: 'background.paper',
    border: '2px solid #000',
    boxShadow: 24,
    p: 4,
    overflow:"scroll"
}

// 改行コード置換（\nを<BR/>に）
const replaceContent = (content) => {
    const str = content.split(/(\n)/).map((item,index) => {
        return (
            <React.Fragment key={index}>
                { item.match(/\n/) ? <br /> : item }
            </React.Fragment>
        );
    });

    return <div>{str}</div>
}

export default function TipsIcon(props) {
    const [open, setOpen] = React.useState(false);
    const handleOpen = () => setOpen(true);
    const handleClose =() => setOpen(false);
    
    return (
        <React.Fragment>
            <Tooltip title="Tips">
               <IconButton
                    size = "small"
                    color = "warning"
                    onClick = {handleOpen}
                    sx = {{marginRight:'10px'}}
                >
                    <IconStyle>
                        <InfoIcon/>
                    </IconStyle>          
                </IconButton>
            </Tooltip>

            <div>
                <Modal
                    open={open}
                    onClose={handleClose}
                    aria-labelledby="modal-modal-title"
                    aria-descrobedby="modal-modal-description"
                >
                <Box sx={ModalStyle}>
                    <Typography id="modal-modal-title" variant="h6" component="h2">
                        <b>{props.title}のTips！</b>
                    </Typography>
                    <Typography id="modal-modal-description" sx={{ mt:2 }}>
                        {replaceContent(props.content)}
                    </Typography>
                </Box>
                </Modal>
            </div>
        </React.Fragment>
    );    
}

\\\\-------------------
datagrid mglist ------
--------------
import React from 'react'
import { DataGrid, GridColDef, jaJP } from '@mui/x-data-grid';
import { Button, Link as MuiLink } from '@mui/material';
import Box from '@mui/material/Box';
import * as common from '../../constants/Common';
import NextLink from "next/link";
import { StyledDataGrid } from "../../styles/Styled";

const DataGrid_list = (props) => {
  const [pageSize, setPageSize] = React.useState<number>(10);

  // 改行コード置換（\r\nを<BR/>に）
  const replaceContent = (content) => {
    const str = content.split(/(\r\n)/).map((item,index) => {
        return (
            <React.Fragment key={index}>
                { item.match(/\r\n/) ? <br /> : item }
            </React.Fragment>
        );
    });
  
    return <div>{str}</div>
  }
   
  const columns: GridColDef[] = [
    { field: 'rowNo', headerName: 'No.', type:'string', width: 70 , headerAlign: 'center', align: 'center', flex: 0.2},
    { field: 'sendDt', headerName: '送信日時', type:'string', headerAlign: 'center', align: 'center', flex: 0.4 },
    { field: 'operatorNm', headerName: '送信先', type:'string', headerAlign: 'center', align: 'left', flex: 0.5 },
    { field: 'urgent', headerName: '至急', type:'string', headerAlign: 'center', align: 'center', flex: 0.2 },
    { field: 'customerNm', 
      headerName: '顧客名', 
      type:'string', 
      headerAlign: 'center',
      align: 'left',
      flex: 0.5, 
      renderCell: (params) => {
        if(!params.row.datFlg) {
          return <NextLink
                  href={{
                    pathname:"BasicRegist",
                    query:{customerCd :params.row.customerCd}
                  }} passHref as = "BasicRegist"
                 >
                 <MuiLink>{params.row.customerNm}</MuiLink>
                 </NextLink>
        } else {
          return params.row.customerNm
        }
    }},
    { field: 'callTypeTxt', headerName: '分類', type:'string', headerAlign: 'center', align: 'left', flex: 0.6 },
    { field: 'memo',
      headerName: 'メモ',
      type:'string',
      headerAlign: 'center',
      align: 'left',
      flex: 0.8,
      renderCell:(params) => {
        return replaceContent(params.row.memo)
    }},
    { field: 'result', headerName: '状態', type:'string', headerAlign: 'center', align: 'center', flex: 0.2 },
    // 隠し項目
    { field: 'customerCd', headerName: '顧客コード', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'seqNo', headerName: '連番', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'urgentFlg', headerName: '至急フラグ', type:'string', headerAlign: 'center', align: 'left', hide: true },
    { field: 'resultFlg', headerName: '完了フラグ', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'callType', headerName: 'コール分類', type:'string', headerAlign: 'center', align: 'left', hide: true },
    { field: 'callClass', headerName: 'コール区分', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'crtDt', headerName: '作成日', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'wrtDt', headerName: '更新日', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'callDt', headerName: 'コール日付', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'callHour', headerName: 'コール時', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'callMin', headerName: 'コール分', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'datFlg', headerName: '論理削除フラグ', type:'string', headerAlign: 'center',align: 'left', hide: true },
    { field: 'defaultFlg', headerName: '初期表示フラグ', type:'string', headerAlign: 'center',align: 'left', hide: true },
  ];

  return (
    <Box
      sx={{
        height: '100%',
        width: '100%',
        // 完了区分（RESULTFLG）が「true：完了」は背景色を変更
        '& .super-app-theme--COMP': {
          bgcolor:common.CST_COLOR_GRAY,
          '&:hover': {
            bgcolor:common.CST_COLOR_HB_GRAY
          }
        },
    }}
    >
      <div>
        <DataGrid getRowId={(row) => row.rowNo}
          sx={StyledDataGrid.grid}
          rows={props.list.map((row) => {
            return {
              ...row,
              rowNo:props.list.indexOf(row)+1
            }
          })}
//          rows={props.list}
          getRowHeight={() => 'auto'}
          columns={columns}
          pageSize={pageSize}
          onPageSizeChange={(newPageSize) => setPageSize(newPageSize)}
          rowsPerPageOptions={[10, 15, 30, 50]}
          pagination
          getRowClassName={(params) => 
            {if(!params.row.resultFlg) {
              return "";
            }
            return `super-app-theme--COMP`}
          }
          localeText={jaJP.components.MuiDataGrid.defaultProps.localeText}
          disableColumnMenu                     // グリッド内のカラムメニュー無効
          disableSelectionOnClick               // 行選択無効
          density='compact'
        />
      </div>
    </Box>
  )
};

export default DataGrid_list;
-----------------------

selectcomponent atoms
\---------------
import React, { useEffect, useState } from 'react';
import { styled } from '@mui/system';
import { FormControl, InputLabel, NativeSelect, Select, MenuItem  } from '@mui/material';
import { SelectProps } from './Type';

const StyleFrom = styled("form") ({
  minWidth: 130
});

export const InputSelect = <T extends {[key:string]:string| number | null}>(props:SelectProps<T>):JSX.Element => {
  const {list,title,indx,column,name,value,unqkey,defaultValue,onChange,options} = props;
  const [selectedValue,setSelectedValue] = React.useState(defaultValue)
  const nonSelected = typeof(defaultValue) === 'string'? '' : 0

  const handleOnChange = (rowData:string) => {
    setSelectedValue(rowData)
    if(list.length === 0) { return }
    console.log(props)
    onChange(indx,column,typeof(defaultValue) === 'string'? rowData : Number(rowData))
  }

  return(
    <FormControl>
      <InputLabel shrink>{title}</InputLabel>
        <StyleFrom>
          <NativeSelect 
            required
            inputProps={{
              name: 'kbn',
              id: 'uncontrolled-native',
            }}
            onChange={e => {handleOnChange(e.target.value as string)}}
            value={String(selectedValue)}
          >
            {options.blank? <option key={0} value={nonSelected}>{''}</option> : ''}
            {
              list?
                list.map((row) => {
                  return(
                    <option key={row[unqkey]} value={String(row[value])}>{row[name]}</option>)
                })
              : <option key={-1} value={0}>{'読込失敗'}</option>
            }
            {options.all? <option key={99} value={nonSelected}>全て</option> : ''}
          </NativeSelect >
        </StyleFrom>
      </FormControl>
  )
};
---------------
opearation  CallClasstype
------------------
import axios from 'axios';
import { createAsyncThunk } from '@reduxjs/toolkit'
import { CallClass } from "./Type";

const url = String(process.env.NEXT_PUBLIC_API_URL);

export const fetchAsyncCallClassList = createAsyncThunk(
  "callClass/list",
  async (value:string) => {
    const res = await axios.get<CallClass[]>(`${url + value}/callClass`)
      .catch(err => {
        return err.response
      });
    if (res.status !== 200) {
      console.log("例外発生時の処理");
      alert('データを取得できませんでした。')
    } else {
      return res.data;
    }
  },
);

---------------
selectore class
-----
import { useSelector } from 'react-redux';
import { RootState } from "../../stores/Store";
import { CallClass } from './Type';

// テナント一覧を取得
export function useCallClassList():CallClass[] {
  return useSelector((state:RootState) => state.callClass.list);
}
-----------------
slice class
--------------
import { createSlice,PayloadAction} from '@reduxjs/toolkit';
import {CallClass,CallClassState} from './Type';
import { fetchAsyncCallClassList } from './Operation';

// コール区分の状態を初期化するよ
const initialCallClassState:CallClassState= {
  list:[] as CallClass[],
  selected:{} as CallClass,
};

const callClassSlice = createSlice({
  name: 'callClass',
  initialState:initialCallClassState,
  reducers: {
     selected(state:CallClassState,action:PayloadAction<CallClass>) {
       state.selected = action.payload
     }
    },
    extraReducers: (builder) => {
      builder
        .addCase(fetchAsyncCallClassList.pending, (state:CallClassState, action:PayloadAction<CallClass[]>) => {
          state.list = action.payload;
        })
        .addCase(fetchAsyncCallClassList.fulfilled, (state:CallClassState, action:PayloadAction<CallClass[]>) => {
          return{
              ...state,
              list: action.payload
          }
        })
        .addCase(fetchAsyncCallClassList.rejected, (state, action) => {
          state.list = [];
        })
    }
});

export default callClassSlice;
------------
type class
------

// あるインターフェースのメンバや型を定義するよ
export interface CallClass{
  [key:string]:string | number | null,
  id:string,
  nm:string
}

export interface CallClassState{
  selected:CallClass,
  list:CallClass[]
}


-------------------mglist Feateure
------------------operation
import axios from 'axios';
import { createAsyncThunk } from '@reduxjs/toolkit'
import { MsgListConditions, MsgListDetail } from "./Type";

const url = String(process.env.NEXT_PUBLIC_API_URL);

const formatDate = (dt) => {
  const yyyy = dt.getFullYear();
  const mm = ('00' + (dt.getMonth() + 1)).slice(-2);
  const dd = ('00' + dt.getDate()).slice(-2);
  return (
    yyyy + '/' + mm + '/' + dd
  )
};

export const fetchAsyncMsgList = createAsyncThunk<MsgListDetail[],MsgListConditions>(
  "msgList/detail",
  async (conditions:MsgListConditions) => {
    // 担当者コード（本人送信分を出す：ログイン担当者、自動送信分を出す：99999999）、表示日付

    // TODO 担当者コードはログイン担当者コードを設定
    var operatorId = "";
    var sendDate = "";

    if(conditions.isShowAuto) {
      operatorId = '99999999';
    } else {
      operatorId = '10000001';
    }

    sendDate = formatDate(conditions.callDate);

    const res = await axios.get<MsgListDetail[]>(`${url}msgList/detail?operatorid=${operatorId}&calldate=${sendDate}`);
    return res.data;
    console.log("log", res.data)
  }

);

-------------------------selectore
\\\\\\\\\^---------------

import { useSelector } from 'react-redux';
import { RootState } from "../../stores/Store";
import { MsgListConditions,MsgListDetail } from './Type';
import { Filter } from "../../helper/Filter";
import { FilterParameter  } from '../../helper/Type';
import { format } from 'date-fns';

// 条件を取得
export function useInputMsgConditions():MsgListConditions {
  return useSelector((state:RootState) => state.msgList.conditions);
}

// 明細を取得
export function useMsgList():MsgListDetail[] {
// テストデータ
//  const list = [
//    {rowNo:"1",callDt:"2022/10/01 10:00",operatorNm:"担当者１",urgent:"●",customerNm:"テスト太郎",callType:"伝言：その他",memo:"伝言状況テスト",result:"",customerCd:"000304406431",urgentFlg:true,resultFlg:false,callClass:"00001",callHour:"10",callMin:"00",datFlg:false,defaultFlg:true},
//  ];

  const conditions = useSelector((state:RootState) => state.msgList.conditions);
  const list = useSelector((state:RootState) => state.msgList.list);


  // filter実行
  var filters:FilterParameter<any>[] = []

  // コール区分コンボボックス未選択時は、以下区分のみ表示させる
  //    00001:伝言、00006:返品理由保留、00013:返品確認、00014:調査依頼
  if(conditions.callClass !== "")
  {
    filters = filters.concat({key:'callClass',column:'callClass',parameter:[conditions.callClass],comparisonFormula:Filter.equal })
  } else
  {
    filters = filters.concat({key:'defaultFlg',column:'defaultFlg',parameter:[true],comparisonFormula:Filter.equal })
  }
  
  // 本日日付の場合は完了分を出す・出さないの制御が可能
  // 本日以外の場合は完了分を出すのみ。
  if(conditions.isToday)
  {
    if(!conditions.isShowComplete)
    {
      filters = filters.concat({key:'resultFlg',column:'resultFlg',parameter:[false],comparisonFormula:Filter.equal })
    }
  }
  
  // 移動ボタン押下時は指定日のみ表示させる
  if(conditions.isMove) 
  {
    filters = filters.concat({key:'crtDt',column:'crtDt',parameter:[format(conditions.callDate,'yyyy/MM/dd').replace(/\//g, "")],comparisonFormula:Filter.equal })
  }

  return Filter.execute(list,filters);
}

--------------------------slice ----------mglist
\\import { createSlice,PayloadAction } from '@reduxjs/toolkit';
import { MsgListConditions,MsgListDetail,MsgListState} from './Type';
import { fetchAsyncMsgList } from './Operation';

const initialMsgListState:MsgListState= {
  list:[] as MsgListDetail[],
  conditions:{
    callDate:new Date(),
    callClass:'',
    operatorId:'',
    isShowComplete:false,
    isShowAuto:false,
    isToday:true,
    isMove:false,
    title:'msgList',
  } as MsgListConditions,
}

// MsgList情報のスライス
const msgListSlice = createSlice({
  name: 'msglist',
  initialState:initialMsgListState,
  reducers: {
    // レデューサーとアクションを生成
     conditionsChanged(state:MsgListState,action:PayloadAction<MsgListConditions>) {
       state.conditions = action.payload
     }
    },
    extraReducers: (builder) => {
      builder.addCase(
        fetchAsyncMsgList.fulfilled,
        (state:MsgListState, action: PayloadAction<MsgListDetail[]>) => {
          return{
            ...state,
            list: action.payload
          }
        }
      )
    }
});

export default msgListSlice;
--------------------

type
----

export interface MsgListConditions{
  [key:string]:string | number | boolean | Date | null,
  title:string,
  callDate:Date,
  callClass:string,
  operatorId:string,
  isShowComplete:boolean,
  isShowAuto:boolean,
  isToday:boolean,
  isMove:boolean
}

export interface MsgListDetail{
  [key:string]:string | number | boolean | Date | null,
  rowNo:string,
  sendDt:string,
  operatorNm:string,
  urgent:string,
  callClassNm:string,
  customerNm:string,
  memo:string,
  result:string,
  customerCd:string,
  seqNo:string,
  urgentFlg:boolean,
  resultFlg:boolean,
  callType:string,
  callClass:string,
  crtDt:string,
  callDt:string,
  callHour:string,
  callMin:string,
  datFlg:boolean,
  defaultFlg:boolean,
}

export interface MsgListState{
  conditions:MsgListConditions,
  list:MsgListDetail[]
}

