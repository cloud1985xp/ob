
請建立一個 user skill， jade-assistant
主要目的是做為 coding assistant，協助彙整理解目前正在開發的項目的進度與狀態


具體行為是

1. 找出目前正在開發的 branch 的 base branch 為何，通常有可能是 develop branch，也有可能是 master or main branch，但請先用 git 的指令去追出最近的一個 base branch，判斷它是從哪個 branch checkout 出來的
2. 經由與 base branch 的 diff 差異比較，理解所有修改的內容，並整理成詳細的筆記，產出在當前專案的 notes/dev/ 下
	1. note 的檔名用當前 branch 名稱為基礎，但做檔名的 normalization，例如將 "/" 替換成 "-"
	2. 若要寫入的 note 檔名已存在，則在檔名後方加上 yyyy-mm-dd 的後綴，例如
	3. 整體路徑例如 notes/dev/{branch-name}-{yyyy-mm-dd}.md
3. 在呼叫這個 skill 時，有可能會同時附帶此次開發內容需求參考資訊，例如功能名稱，或開發內容的規格、文件；如果有提供的時候，請務必先理解其內容，再進行對程式碼的比較、理解與進度彙整
4. 過程中不去更改任何的程式碼