我想要先重整並建立系統中的共用 UI 元件
用新的前端 stack 下把系統中各種元件先從舊版本移植到新版本

讓我們一步一步來

首先是有關 tags 與 contacts 的 input / select 欄位

請先閱讀 feature/api 與 feature/t2646-selection-api  兩個 branch 間的差異
這段修改主要在舊版 stack 中，將 tags / contacts 使用的 Tagify 套件做一層包裝
為成一個共用元 `TagifyWithApiSearch` 
並讓它擁有使用 api 作動態查詢選項的模式，同時給 tags / contacts 使用

我想要將這個類似的設計移植到新版的前端 stack 中

主要是特色有：
- 元件的參數都綁定在該 input 元素的 data-attributes 上
- 依參數決定是否啟用 api 查詢模式

目前現在這個 branch 上已有部分新的 ui component 的製作，請參考 2f8432463764b7c6b5f9f83babb6b9919c569e1e 這個 commit

該 commit 在新版 stack 中改用了 tom_select 這個套件做為基底
建立了 contact_select_controller, 與 contact_select_controller
請幫我修改這兩支 select 元件，讓它也可以像舊版的 TagifyWithApiSearch 具有上述的兩種特色

請先調整 js 就好，實際套用的部分我們後面再進行