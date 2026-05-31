# 專案說明

消消樂這款益智遊戲的目標，主要是消除同個顏色的方塊直到板面淨空；方法即是要點擊兩個方格在直線間的"連續空白方格" (其一即可) ，或者是點擊兩個同顏色方格在行列直角相交處的空白方格 (也需連續空白)

<img width="621" height="420" alt="螢幕擷取畫面 2026-04-07 021712" src="https://github.com/user-attachments/assets/329ed8ee-5513-40af-b2e5-6d885b7ea2c2" />

回顧Proposal 的預期功能；我在這個專案裡面的目標是: 

1. **可以不需要紀錄解題路徑便可以判別地圖是否有完美解的演算法** : 當我輸入一個隨機生成的地圖時 (包含遊戲內目前所知的所有元素，但是每種顏色格子的擺放方式與數量皆不盡相同，測試資料都是由亂數決定的結果)，演算法可以在不直接確認執行步驟下，
推論出這張地圖的所有方格是否可以只透過遊戲內指定的兩種方式去全數消除 (也就是說，我只讀入顏色方格內容，但是電腦不需要實際操作到最後全部消除；只要純看地圖內容就知道是否有解)。
此演算法難在理論證明，因為我需要能依靠個人能力證明一張地圖有解的特質 (與為甚麼)，這樣就只需將這些特質作為條件確認，輸出 "有解" 或 "無解" (顯而易見的測試方式，就是至少當我直接依照正式遊戲內的既定地圖去輸入地圖情形；因為外面網站中的遊戲地圖
保證有解，那因此這個演算法理當輸出 "有解")。

2. **尋找一個可行路徑的演算法** : 若我無法在時間內證明一張地圖是否可以在不實際執行操作的情況下確認有無完美解 (或是根本不存在我理想中的萬用確認方式)，那第二種演算法就是去實際跑很多遍解題流程，只依靠前述兩種方式依序消除方塊嘗試解題，
直到最後確認結果。由於這個遊戲的方格很多，可能運算的時間花費極大才能找到一個解。因此這時候我目前**預期中的處理方式**有兩種: 1. **減少格子總數** (對目標達成來講比較好) 2. **限制運算時間** (可能會在某些情況無法達成目標，因為可能有解但是演算法能力
不足無法在限制時間內找到，就跟人腦一樣)。第二種演算法必定會實作；原因是若我推導出第一種演算法的有解證明條件，那我也需要透過路徑搜尋的演算法去進行確認，證明兩種演算法在理論上與實務上都可以導出有解的結果。

以下是針對兩項目標最後獲得的結果討論，以及程式碼的運算內容比較: 

**項目一**
我在Prototype Report 中已有提到很多針對地圖不可解的特性討論以及地圖大小對可解性的影響 (這裡不重複補上，但是討論內容都很重要請觀看 !)；最終我定案: **專案第一項目標不存在不使用搜尋演算法即可判斷是否有解的方式**。
我們可以再將這個專題遊戲與魔術方塊一起討論:

魔術方塊的解題方法，在本質上是不變量檢查，而本遊戲的可解性，很依賴對每次動態消除過程中地圖變化的觀察 (格子消除完，可點擊的範圍也更大；但是魔術方塊元素永久不變)。魔術方塊只要透過靜態觀察特定顏色方塊的相對位置，如角塊與邊塊的朝向，就能利用數學公式在不實際轉動方塊的情況下確認題目是否可解(畢竟魔術方塊邊角若被旋轉，可能影響到可解性)以及找出實際解題步驟，並直接套用固定的公式步驟得出解答，相反地，本遊戲無法只憑藉初始盤面的方塊數量或分布(每個顏色格子之數量與可以用來點擊消除的空白格子在位置與數量會因地圖型態不同)，就給出可解與否的結論。只有模擬每一種點擊後的未來狀態，也就是透過 A* 或 DFS 搜尋演算法，才能達成我們的目的。若真的要舉例，那就是在Prototype Report提到的一個格子四面都被其他方塊包圍；我們也一定需要知道那四面的其他方塊是否能被消除，如此遞迴下去就又回歸到路徑搜尋，這樣就不是通用公式找解了。

再者，從可逆性與不可逆性的角度來看，魔術方塊的任何轉動都是可逆的，轉順時針就能轉逆時針復原，這在數學上形成了一個群，因為具備可逆性，狀態與狀態之間是雙向連通的，你可以從任意狀態推導出與目標狀態的"目標距離"，但這個消消樂遊戲的點擊消除是絕對不可逆的，一旦為了消除某個顏色而點擊了某個空格，那幾顆方塊就被移除，無法在遊戲規則內恢復原狀(程式碼的運算過程也不是把這些方格"恢復"，而是倒回到上一個還沒行動的時機點)，這種不可逆性導致遊戲進程是一個有向無環圖，走錯一步就導致不能完美消除，使得我們無法像魔術方塊一樣用多組通用的靜態公式去套用解題。

**項目二**
針對項目2裡面進行的處理方式；最後在實際製作搜尋演算法時，我採用的方式是減少格子總數。在本次專題我採用的兩種演算法分別是 DFS Backtracking 和 A* Search，DFS Backtracking 運作時，僅是在50個可消除格子的地圖中就會需要計算5~6分鐘，時間花費較多。
減少格子數量也有助於我分別對兩種演算法執行更多次測試，來比較具有不同性質的地圖會讓搜尋演算法有多少使用時間上的差異 (以及背後的原因)。最終定案在50個可消除格子的地圖去討論。

在進入運算結果討論之前，我需要先補充Prototype Report一個未被提到的內容: 三消的可能性 (一次消除三格) 。
當2n種顏色的格子數為奇數，那就代表這些格子必須要可以被三消，否則一種顏色在兩格兩格消除過後會只剩下一格；地圖無法被完美解。三消的條件在於，一個顏色需有三個方塊的分布為T字型；意即兩個可以直線消除的方塊彼此間隔內需要有一個空白方格，
其垂直方向也有一個同色方格，這樣子點擊三個同色方格的垂直交界處就可以完成三消。如果具有奇數個格子的某顏色方塊，每三格皆完全不具有一種T字型分布，就代表不可解。

將這個因素討論進去以後，開始實作程式碼。我透過輸入自行設計的可解地圖 (先測試是否演算法導出的答案真的可以完全消除地圖上格子)，以及AI幫我生成的完全隨機地圖 (測試演算法是否能正確推論一個地圖是否不可解)進行測試，獲得以下結果與時間紀錄:

**1. 一個保證有解(我先行設計)；且每種顏色皆為偶數格的地圖 (50格)**

<img width="1008" height="167" alt="螢幕擷取畫面 2026-05-31 161910" src="https://github.com/user-attachments/assets/b7e58d93-d545-453b-8641-744b9c8a73b6" />


**(左: DFS 版本，時間花費1秒以內 右: A-star 版本，時間花費一秒以內)**


<img width="326" height="615" alt="螢幕擷取畫面 2026-05-31 041708" src="https://github.com/user-attachments/assets/11032995-8d8a-4c60-b3ed-257f34ce2965" />

<img width="301" height="594" alt="image" src="https://github.com/user-attachments/assets/967bab18-278c-4c96-9358-f58003803acc" />

**2. 一個保證有解(我先行設計)；且具有三消格式的地圖 (50格)**

<img width="1011" height="166" alt="螢幕擷取畫面 2026-05-31 161940" src="https://github.com/user-attachments/assets/22c26242-f258-4dc7-bde6-20b541051238" />


**(左: DFS 版本，時間花費3分鐘左右 右: A-star 版本，花費時間五秒左右)**


<img width="335" height="589" alt="螢幕擷取畫面 2026-05-31 042941" src="https://github.com/user-attachments/assets/190242f3-be23-4f0a-bf9c-2a713812eed9" />

<img width="449" height="534" alt="螢幕擷取畫面 2026-05-31 043133" src="https://github.com/user-attachments/assets/a475a21c-ac4c-4466-b75a-65156cd4f4f7" />



**3. AI 生成的隨機地圖 (50格)**
**(地圖一；左: DFS 版本，時間花費30秒左右 右: A-star 版本，花費時間5秒左右)**

<img width="714" height="162" alt="螢幕擷取畫面 2026-05-31 152338" src="https://github.com/user-attachments/assets/e3bb441e-65e2-4887-b375-fc744852c0ee" />

<img width="450" height="589" alt="螢幕擷取畫面 2026-05-31 143454" src="https://github.com/user-attachments/assets/e03f24c8-8bc2-47ed-b8b7-15534d5d7bc5" />

<img width="458" height="542" alt="螢幕擷取畫面 2026-05-31 150714" src="https://github.com/user-attachments/assets/b92e2513-fe66-46e1-821b-9cefdf0b9fdd" />



**(地圖二；左: DFS 版本，時間花費10秒左右 右: A-star 版本，花費時間2秒左右)**

<img width="1011" height="169" alt="螢幕擷取畫面 2026-05-31 152359" src="https://github.com/user-attachments/assets/e73aa003-f6b6-40fb-ba07-66c5da3e3999" />

<img width="324" height="567" alt="螢幕擷取畫面 2026-05-31 145010" src="https://github.com/user-attachments/assets/827eda77-0381-4004-815c-dd7543a7a3e2" />

<img width="370" height="540" alt="螢幕擷取畫面 2026-05-31 150315" src="https://github.com/user-attachments/assets/ababcb39-ed4d-47e5-ba86-fd4ed1786f94" />



**(地圖三；左: DFS 版本，時間花費1分鐘左右 右: A-star版本，花費時間30秒左右)**


<img width="1009" height="187" alt="螢幕擷取畫面 2026-05-31 152419" src="https://github.com/user-attachments/assets/566a44ac-3dec-4aeb-bca8-ad40a5205cd7" />

<img width="306" height="575" alt="螢幕擷取畫面 2026-05-31 145705" src="https://github.com/user-attachments/assets/41d26fc7-919f-45a6-b299-e67cf1575c90" />

<img width="326" height="587" alt="螢幕擷取畫面 2026-05-31 145943" src="https://github.com/user-attachments/assets/cd6b84e5-64c5-49fd-8753-972aabbcf2e9" />


為何在時間花費上會有差異? 我們就兩種演算法的策略進行討論:
1. DFS Backtracking: 在連續性步驟題型中，DFS Backtracking基本上就是盲目搜尋；持續在遇到無解情況時嘗試回溯，持續遞迴到成功抵達終點，或是所有可能性都被窮舉完畢，但仍無解。這種過程缺乏外在引導，因此在格子數越大、排序較複雜的時候，花費時間會大幅增加。
2. A* Search: 這種演算法會開設兩個集合: Open List (未探索資訊；在專題內就是"未來可以嘗試消除方格的路線"，會將此步驟進行函式計算並與其他種步驟進行比較)，以及Closed List(儲存已經被探索的路徑狀態，以確保分析過後的步驟不再重新被考慮)。對於Open List 中列出的所有可能步驟我們會設計一個成效分數 : nextNode.f = nextNode.g + 5 x (nextNode.h) (g代表已經走了幾步，h代表至少還要再走幾步；這個成效分數以未來步驟為焦點，因此我給h加了五倍的權重；期望計算機尋找一個能盡可能減少未來步數的路線)。因為我們有為這個程式提供一個**外部的指引函數** (目標即是要讓已經走的步驟+距離終點預估值越小越好)，進入"單一個"錯誤分支時會直接進行修剪而非走到完全無解的路線 (例如；設一種顏色只剩下五格，代表需要三消，DFS一開始看到了可以兩消的其中兩格就直接行動；這樣會產生一次無用的運算路徑。但是A* Search 會保留這個可以一次消除三格，加速步驟執行的行動，正好符合正確的路徑，因此更快求出單個正確解答。)

針對結果輸出步驟的差異，重點在於我為A* 設計的f(n)函式較偏好可以減少未來操作步驟數；因此可以看到A* 採用的路徑會較多包含可以一次消除兩對 (四格) 的路徑組合。但是DFS會直接輸出 "第一個" 他找到可以完全解的路徑，因此不僅時間花費可能更多，解題所需的步驟數也可能較多。

DFS的時間複雜度在Worst Case 是 O(n^d) (d是步驟數)，而A* Search在 f(n) 足夠優秀的情況下可以將無效分支剪除，進而大幅度壓低運算範圍。因此，在50格地圖格子分布更加複雜的情況下(三消運算更多時)，DFS和A* Search 的運算效率就已經產生了很大幅度的差距。

這兩種演算法在實務運作上的差異，成功在本專案產生了不同的效果 (就時間複雜度而言)

# 使用方式 (與程式碼部分內容的詳盡討論)

使用方式: 將設計好的地圖用以下格式依序輸入:
0 0 y (Row, Column, color)

0 1 g

0 2 o

0 4 b

0 5 r

0 6 y

..... (直到五十個格子輸入完畢)

輸入完畢之後，系統會先跳出總格子數以及每個顏色分別的格子數。接著開始運算路徑，若有解則依序生成；若無解則生成 "No feasible answer for this map."

接下來是部分程式碼核心內容的詳細註釋:

**1. 在一個地圖狀態中，尋找每個可以消除任意顏色方塊的空格**

vector<vector<BlockInfo>> getAllValidMoves() {
    vector<vector<BlockInfo>> moves; // 用來儲存所有找到的合法步法

    // 雙層迴圈遍歷整張地圖的每一個格子
    for (int r = 0; r < MAX_ROW; ++r) {
        for (int c = 0; c < MAX_COL; ++c) {

            // 只有當當前格子是「空位 (0)」時，才能以此為中心向外搜尋其他格子的顏色
            if (gameMap[r][c] == 0) {
                // 初始化上、下、左、右四個方向第一個撞到的方塊座標，預設為 -1 表示沒撞到
                Point p_up = { -1, -1 }, p_down = { -1, -1 }, p_left = { -1, -1 }, p_right = { -1, -1 };

                // 往上搜尋第一個不是 0 的方塊
                for (int i = r - 1; i >= 0; --i) if (gameMap[i][c] != 0) { p_up = { i, c }; break; }
                // 往下搜尋第一個不是 0 的方塊
                for (int i = r + 1; i < MAX_ROW; ++i) if (gameMap[i][c] != 0) { p_down = { i, c }; break; }
                // 往左搜尋第一個不是 0 的方塊
                for (int j = c - 1; j >= 0; --j) if (gameMap[r][j] != 0) { p_left = { r, j }; break; }
                // 往右搜尋第一個不是 0 的方塊
                for (int j = c + 1; j < MAX_COL; ++j) if (gameMap[r][j] != 0) { p_right = { r, j }; break; }

                // 將四個方向有找到的有效方塊放進可視清單中
                vector<Point> visible;
                if (p_up.r != -1) visible.push_back(p_up);
                if (p_down.r != -1) visible.push_back(p_down);
                if (p_left.r != -1) visible.push_back(p_left);
                if (p_right.r != -1) visible.push_back(p_right);

                // 用來儲存從當前這個空位出發，所有可能產生的消除組合
                vector<BlockInfo> currentClickMoves;

                // 檢查 1 到 5 號每一種顏色
                for (int color = 1; color <= 5; ++color) {
                    vector<Point> colorGroup;
                    // 在四個方向看得到的方塊中，找出屬於當前顏色的方塊
                    for (Point p : visible) {
                        if (gameMap[p.r][p.c] == color) {
                            colorGroup.push_back(p);
                        }
                    }
                    // 如果同一個顏色出現了 2 個或以上，代表符合消除規則
                    if (colorGroup.size() >= 2) {
                        for (Point p : colorGroup) {
                            currentClickMoves.push_back({ p, color });
                        }
                    }
                }

                // 如果當前這個空位有合法的消除路徑，將其排序後加入currentClickMove
                if (!currentClickMoves.empty()) {
                    sort(currentClickMoves.begin(), currentClickMoves.end());
                    moves.push_back(currentClickMoves);
                }
            }
        }
    }

    // 將所有收集到的步奏進行排序，確保演算法不重複排同個步驟
    sort(moves.begin(), moves.end());
    moves.erase(unique(moves.begin(), moves.end()), moves.end());

    return moves;
}

**2. DFS Backtracking 運算函式**

bool dfs() {
    if (currentblocks == 0) return true; // 盤面全清空確認有解


    string state = "";
    for (int r = 0; r < MAX_ROW; ++r) {
        for (int c = 0; c < MAX_COL; ++c) {
            state += (char)('0' + gameMap[r][c]);
        }
    }
    // 如果這個盤面之前走過且確實無解，直接回溯
    if (visitedStates.count(state)) return false;
    visitedStates.insert(state); // 紀錄這個盤面已經來過了

    // 不可解之後剪枝：若地圖上某種顏色剩餘數量剛好為 1，表示無法消除並立刻剪枝回溯
    int colorCounts[6] = { 0 };
    for (int r = 0; r < MAX_ROW; ++r) {
        for (int c = 0; c < MAX_COL; ++c) {
            colorCounts[gameMap[r][c]]++;
        }
    }
    for (int i = 1; i <= 5; ++i) {
        if (colorCounts[i] == 1) return false;
    }

    // 獲取當前盤面所有的合法操作 (即空白組合)
    vector<vector<BlockInfo>> moves = getAllValidMoves();
    if (moves.empty() && currentblocks > 0) return false; // 無子可動，死局回溯

    for (const auto& move : moves) {
        // 1. 執行消除
        removeBlocks(move);
        currentblocks -= move.size();

        // 2. 遞迴搜尋
        if (dfs()) return true;

        // 3. 失敗則回溯：地圖還原狀態、退出 Stack
        currentblocks += move.size();
        restoreBlocks();
    }
    return false;
}

**3. A-star Search 函式**

bool aStarSearch() {
    // 建立Priority queue，f 值"越小"的盤面狀態會越先處理
    priority_queue<AStarNode, vector<AStarNode>, greater<AStarNode>> pq;

    // 1. 初始化起始節點
    AStarNode startNode;
    startNode.history = {};                   // 歷史紀錄初始化
    startNode.currentblocks = currentblocks;   // 記錄"目前"的方塊總數
    startNode.g = 0;                          // 起始步數為 0
    startNode.h = currentblocks / 2;          // 起始預估值：剩餘方塊除以 2（最理想狀況下需要的步數）
    startNode.f = startNode.g + 5 * startNode.h; // 計算總權重（加強貪婪比重，引導演算法快速找答案）

    // 將全域的初始地圖複製到起始節點的棋盤中
    for (int r = 0; r < MAX_ROW; ++r)
        for (int c = 0; c < MAX_COL; ++c)
            startNode.board[r][c] = gameMap[r][c];

    // 將起始節點放入佇列中
    pq.push(startNode);

    // 將起始地圖狀態轉換為字串，並標記為「已造訪」，避免重複搜尋
    string startState(MAX_ROW * MAX_COL, '0');
    int idx = 0;
    for (int r = 0; r < MAX_ROW; ++r)
        for (int c = 0; c < MAX_COL; ++c)
            startState[idx++] = '0' + gameMap[r][c];
    visitedStates.insert(startState);

    // 開始進行 A* 迴圈搜尋
    while (!pq.empty()) {
        // 取出當前評估最優（f 值最小）的節點
        AStarNode curr = pq.top();
        pq.pop();

        // 【成功條件】如果地圖上已經沒有任何方塊，代表成功解出！
        if (curr.currentblocks == 0) {
            moveStack = curr.history; // 將過關的步驟紀錄複製給全域變數
            return true;
        }

        // 2. 還原全域地圖
        for (int r = 0; r < MAX_ROW; ++r)
            for (int c = 0; c < MAX_COL; ++c)
                gameMap[r][c] = curr.board[r][c];
        currentblocks = curr.currentblocks;

        // 3. 取得當前盤面下所有合法的消除步奏
        vector<vector<BlockInfo>> moves = getAllValidMoves();

        // 遍歷每一條可行的分支
        for (const auto& validMove : moves) {
            // 【模擬】在全域地圖上將選定的一組方塊消除（設為 0）
            for (const auto& bi : validMove) {
                gameMap[bi.p.r][bi.p.c] = 0;
            }

            // 將消除後的「新地圖」轉換為字串狀態
            string nextState(MAX_ROW * MAX_COL, '0');
            int sIdx = 0;
            for (int r = 0; r < MAX_ROW; ++r) {
                for (int c = 0; c < MAX_COL; ++c) {
                    nextState[sIdx++] = '0' + gameMap[r][c];
                }
            }

            if (visitedStates.count(nextState) == 0) {

                //計算地圖上各個顏色的剩餘數量
                int colorCounts[6] = { 0 };
                for (char ch : nextState) {
                    colorCounts[ch - '0']++;
                }
                bool unsolvable = false;
                // 只要有任何一種顏色「只剩下 1 個方塊」，這個盤面就絕對無法全清了，判定無完美解
                for (int i = 1; i <= 5; ++i) {
                    if (colorCounts[i] == 1) { unsolvable = true; break; }
                }

                // 如果盤面依然有解，才將新節點推入 Priority Queue
                if (!unsolvable) {
                    visitedStates.insert(nextState); // 標記此狀態已造訪

                    AStarNode nextNode;
                    // 將模擬消除後的盤面複製給新節點
                    for (int r = 0; r < MAX_ROW; ++r)
                        for (int c = 0; c < MAX_COL; ++c)
                            nextNode.board[r][c] = gameMap[r][c];

                    nextNode.history = curr.history;              // nextnode繼承歷史步驟
                    nextNode.history.push_back({ validMove });     // 加入本次步驟
                    nextNode.currentblocks = curr.currentblocks - validMove.size(); // 更新方塊剩餘數
                    nextNode.g = curr.g + 1;                      // 實際消耗步數 + 1
                    nextNode.h = nextNode.currentblocks / 2;      // 計算新盤面的啟發預估值
                    nextNode.f = nextNode.g + 5 * nextNode.h;     // 計算新盤面的總分

                    pq.push(nextNode); // 推入優先佇列中等待後續搜尋
                }
            }

            //不論該分支被放入佇列還是被剪枝，都必須還原地圖，以便測試下一個合法步法
            for (const auto& bi : validMove) {
                gameMap[bi.p.r][bi.p.c] = bi.color;
            }
        }
    }
    // 佇列空了卻還沒觸發 currentblocks == 0，代表此題完全無解
    return false;
}
