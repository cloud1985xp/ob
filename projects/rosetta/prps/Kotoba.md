
LQA/Planner 整理好 LQA Sheet 提出 request

- chara_sheet
	- Rosetta 依 subject 分組，篩選出需要採用 kotoba 翻譯的 subjects，對每個 subjects
		- 產生 temporate_sheet
		- 執行 generate request
			- 呼叫 generate_mt5_test_ishin
		- 完成 generate 後，將翻好的資料送回 rosetta
	- Rosetta 產生翻譯包
		- 對翻譯好的內容，使用 AI correction
		- 可對不滿意或一開始就標記的資料，採用 AI translation
		- 產生 AI 校閱、評分
		- 下載翻譯包

