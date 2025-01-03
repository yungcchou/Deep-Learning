# 2D Convolutional Neural Network (2D 卷積神經網路)

**卷積運算** (Convolution)，特別是**卷積神經網路**（CNN力Convolutional Neural Network），在各種**深度學習**任務中被廣泛應用。
它們在處理**空間數據**（如影像、視訊，甚至某些類型的時間序列數據）方面特別有效。
在與**全連接神經網路**的比較中，卷積的優勢可以分為幾個重要方面:

1. **局部感受域 (Local Receptive Fields)（獲取空間層次結構）**
   - 卷積透過關注數據的局部區域來捕捉**空間層次結構**和**局部模式**。
   - 對於影像，這意味著模型可以在早期層識別**邊緣**、**紋理**或**簡單形狀**，在更深層次識別更複雜的結構（如物體）。
2. **參數效率（權重共享）**
   - 在全連接網路中：每個神經元與每個輸入特徵相連接，這會導致大量的參數，特別是對於像影像這樣的大輸入（例如，$28\times28$ 的影像有 784 個像素）。
   - **在 CNN 中**：一個單一的過濾器在輸入上滑動（空間上或時間上），並在每個位置應用相同的權重。這顯著減少了參數的數量。
3. **平移不變性**
   - 卷積網路可以無論特徵出現在輸入的何處都能檢測到它們。如果一個特徵（如邊緣或模式）出現在影像的不同位置，卷積過濾器可以在任何地方識別它。
4. **層次化特徵學習**
   - 在 CNN 中，靠近輸入的層學習檢測低級特徵，如邊緣、角點和紋理，而較深層次則捕捉更高級的特徵，如形狀、模式和物體。
   - 這種層次結構允許 CNN 建立越來越抽象的輸入表示。
   - **在視覺任務中**：第一層可能檢測邊緣或小紋理，而後面的層則檢測人臉、動物或整個物體。
5. **處理高維數據**
   - 卷積層通過池化和卷積運算逐步減少輸入數據的維度，從而在不丟失重要空間信息的情況下，更容易處理高維數據，如大影像或視訊。
   - 池化層：這些層透過下採樣（例如最大池化）來降低特徵圖的解析度，從而減少計算成本，同時保留最重要的特徵。
6. **更少的參數需要學習**
   - 對於大輸入，全連接層因參數激增而迅速變得難以處理，但 CNN 透過權重共享學習的參數更少。這減少了過擬合的風險並加快了訓練速度。
   - 例如：對於大小為 $28 \times 28 \times 3$（RGB 通道）的影像，全連接層僅一層就需要 $784 \times 3 \times N$ 個參數，其中 $N$ 是神經元的數量。而卷積層使用大小為 $3 \times 3$ 的濾波器，則每個濾波器只需要 $3 \times 3 \times 3 = 27$ 個參數。
7. **結構化數據：網格狀拓撲結構**  
   - 卷積運算特別適合處理網格狀的**拓撲結構**，例如影像（2D 網格）或視訊（3D 網格，時間為第三維度）。  
   - 它們能夠在局部區域內**滑動**並捕捉模式，使其成為處理這類數據的理想選擇。  
   - **應用領域**：卷積通常應用於影像處理（如**物體檢測**、**分割**）、**視訊分析**以及某些 1D 任務，如時間序列預測（在這種情況下可以使用 1D 卷積）。

8. **正則化：減少過擬合**  
   - 通過使用**小濾波器**和**局部連接**，CNN 對學習過程施加了強烈的**先驗約束**。這可以**減少過擬合**的風險，尤其是在與可能通過學習遙遠輸入間的偶然相關性而過擬合的全連接網絡相比時。

9. **比例與變形不變性（使用先進架構）**  
   - 雖然基本的 CNN 可以捕捉**平移不變性**，但一些先進技術，如**空間變換網絡**和**膨脹卷積**，讓 CNN 在面對輸入數據的**比例**、**旋轉**或**變形**變化時更具強韌性。  
   - **應用領域**：在物體檢測任務中，當物體以不同比例或方向出現在影像中時，這些技術幫助 CNN 更好地適應。

---

## The core procedures of CNNs

CNN的核心過程涉及透過卷積層、池化層和全連接層提取層次化的特徵。以下是CNN過程的具體說明：

### 1. **輸入數據**

對於CNN，輸入數據通常是多維的：

- **圖像**：一個2D或3D的像素值陣列。
  - 對於灰度圖像：輸入大小為 $H \times W \times 1$ （高度，寬度，1個通道）。
  - 對於RGB圖像：輸入大小為 $H \times W \times 3$ （高度，寬度，3個通道）。

給定一個高度為 $H$、寬度為 $W$ 並有 $C$ 個通道的圖像，輸入可以表示為：
$$X = \{x_{h,w,c}\}, \quad h \in [0, H-1], w \in [0, W-1], c \in [0, C-1]$$

其中， $x_{h,w,c}$ 是在位置 $(h, w)$ 通道 $c$ 上的像素值。

### 2. **卷積層**

卷積層使用濾波器（或稱為內核）來提取局部特徵。濾波器是一個小型的權重矩陣，在輸入上滑動並在每個位置計算點積。

- **卷積操作**：對於2D卷積，給定輸入 $X$ 和濾波器 $F$，卷積操作會生成一個特徵圖 $Y$。假設輸入的尺寸為 $H \times W \times C$，濾波器的尺寸為 $k_H \times k_W \times C$（濾波器的高度、寬度和深度）。

在每個位置 $(h, w)$，卷積輸出的計算方式為：

$$Y_{h, w} = \sum_{i=0}^{k_H-1} \sum_{j=0}^{k_W-1} \sum_{c=0}^{C-1} F_{i,j,c} \cdot X_{h+i, w+j, c}$$

其中 $F$ 是濾波器內核，輸出 $Y$ 是生成的特徵圖。該操作在整個輸入上重複進行，根據指定的步幅滑動濾波器。

- **步幅**：步幅控制濾波器在輸入上移動的方式。步幅為1表示濾波器每次移動1個像素，而較大的步幅表示濾波器每次移動更多個像素。
  
- **填充**：填充用於控制輸出的大小。填充 $p$ 會在輸入周圍添加零邊框：
  - **Valid填充**（無填充）：輸出尺寸縮小。
  - **Same填充**（零填充）：輸出尺寸與輸入相同。

### 3. **激活函數 (ReLU)**

在應用卷積操作後，會應用激活函數（通常是ReLU：修正線性單元）來引入非線性。ReLU定義為：

$$f(x) = \max(0, x)$$

這個操作將特徵圖中的所有負值替換為零，幫助網絡建模非線性關係。

### 4. **池化層**

池化層減少特徵圖的空間維度（高度和寬度），同時保留最重要的信息。這有助於減少計算複雜度並防止過擬合。

- **最大池化**：最常見的池化操作是最大池化，在每個由池化濾波器覆蓋的區域中取最大值。如果我們使用 $2 \times 2$ 的池化窗口，且步幅為2，則輸出為：

$$Y_{h, w} = \max(X_{h+i, w+j}), \quad i, j \in [0, 1]$$

其中， $Y$ 是下採樣後的特徵圖， $X$ 是輸入特徵圖。

- **平均池化**：取濾波器覆蓋區域的平均值，而非最大值。

### 5. **全連接層（密集層）**

經過一系列的卷積和池化層後，生成的特徵圖會被展平為一個1D向量，然後傳遞到全連接層，進行最終的分類。

令展平後的輸入向量為 $z$。全連接層的計算方式為：

$$y = Wz + b$$

其中：

- $W$ 是權重矩陣。
- $b$ 是偏置向量。
- $y$ 是輸出（對數機率）。

### 6. **輸出層（Softmax）**

CNN的最後一層通常是用於多分類的 Softmax 層。Softmax函數將對數機率轉換為概率，確保所有類別的概率總和為1：

$$P(y=k) = \frac{e^{z_k}}{\sum_{i=1}^{K} e^{z_i}}$$

其中 $z_k$ 是類別 $k$ 的對數機率， $K$ 是總類別數。

### 7. **損失函數**

對於分類任務，常用的損失函數是分類交叉熵：
$$L(y, \hat{y}) = -\sum_{i=1}^{K} y_i \log(\hat{y}_i)$$
其中：

- $y_i$ 是真實標籤（獨熱編碼）。
- $\hat{y}_i$ 是類別 $i$ 的預測概率。

### 8. **反向傳播與優化**

一旦完成CNN的前向傳播並計算出損失，模型的權重會使用 **反向傳播** 和優化演算法（如 **隨機梯度下降** 或 **Adam**）來更新。

- **反向傳播** 利用鏈式法則計算損失函數對每個權重的梯度。
- **優化器** 通過更新權重來最小化損失。

### CNN操作的關鍵步驟總結

1. **卷積**：透過濾波器提取局部特徵。
2. **激活**：引入非線性（通常是ReLU）。
3. **池化**：降低空間維度，同時保留最重要的信息。
4. **展平**：將2D特徵圖轉換為1D向量。
5. **全連接層**：綜合所有特徵並進行分類。
6. **Softmax輸出**：將對數機率轉換為類別概率。
7. **反向傳播與優化**：最小化損失並更新權重。

### 一個簡單的CNN圖像分類架構範例

1. 輸入： $28 \times 28 \times 1$（灰度圖像）。
2. 卷積層 1：32個 $3 \times 3$ 濾波器。
3. ReLU激活。
4. 最大池化： $2 \times 2$ Window。
5. 卷積層 2：64個 $3 \times 3$ 濾波器。
6. ReLU激活。
7. 最大池化： $2 \times 2$ window。
8. 展平層：將2D特徵圖展平為1D。
9. 全連接層（密集層）：128個單元。
10. 輸出層（Softmax）：10個單元，用於10個類別的分類。

這個過程使CNN能夠高效地學習空間層次結構和模式，特別適合於圖像、視頻以及結構化數據的處理。

---

## 數位影像處理常用的各種 Kernel

在 2D 卷積神經網路（CNN）中，設計用於圖像特徵提取的卷積核（Kernel）通常包括一些標準的濾波器，用來檢測邊緣、紋理和形狀等特徵。以下是一些常見的 2D CNN 卷積核，通常用於圖像特徵提取：

### 1. **邊緣檢測卷積核**

- 在**圖像邊緣檢測**中，使用不同的卷積核設計可以**強調**各種**邊緣方向**和**細節**。
- 邊緣檢測卷積核可以強調圖像中的**邊緣**或**輪廓**$，這在**目標檢測**和**識別**任務中非常關鍵。
- 當使用 $5 \times 5$ 和 $7 \times 7$ 的卷積核進行邊緣檢測時，**較大的卷積核**可以捕捉**更廣泛**的圖案和**圖像中的大尺度特徵**。
- 較大的卷積核能夠**檢測更細微**的像素強度變化，因此適合用來回應**更寬**的**邊緣**或**更複雜的紋理**。
以下是一些常用於圖像邊緣檢測的 2D 卷積核，每個核都針對特定的邊緣類型：

#### 1.1 **Sobel 運算子**

- Sobel 運算子是邊緣檢測中常用的工具，提供了檢測垂直和水平邊緣的不同卷積核。
- 這些卷積核可以有效地檢測圖像中的**水平**和**垂直邊緣**，強調**像素值**的**急劇變化**。

- **水平邊緣檢測 ($3\times3$)**：

$$
\begin{bmatrix}
-1 & 0 & 1 \\
-2 & 0 & 2 \\
-1 & 0 & 1 \\
\end{bmatrix}$$

- **水平邊緣檢測 ($5\times 5$)**：

$$
\begin{bmatrix}
-1 & -2 & 0 & 2 & 1 \\
-4 & -8 & 0 & 8 & 4 \\
-6 & -12 & 0 & 12 & 6 \\
-4 & -8 & 0 & 8 & 4 \\
-1 & -2 & 0 & 2 & 1 \\
\end{bmatrix}$$

- **水平邊緣檢測 ($7\times 7$)**：

$$
\begin{bmatrix}
-1 & -2 & -3 & 0 & 3 & 2 & 1 \\
-4 & -8 & -12 & 0 & 12 & 8 & 4 \\
-5 & -15 & -20 & 0 & 20 & 15 & 5 \\
-6 & -24 & -30 & 0 & 30 & 24 & 6 \\
-5 & -15 & -20 & 0 & 20 & 15 & 5 \\
-4 & -8 & -12 & 0 & 12 & 8 & 4 \\
-1 & -2 & -3 & 0 & 3 & 2 & 1 \\
\end{bmatrix}$$

- **垂直邊緣檢測 ($3\times3$)**：

$$
\begin{bmatrix}
-1 & -2 & -1 \\
0 & 0 & 0 \\
1 & 2 & 1 \\
\end{bmatrix}$$

- **垂直邊緣檢測 ($5\times 5$)**：

$$
\begin{bmatrix}
-1 & -4 & -6 & -4 & -1 \\
-2 & -8 & -12 & -8 & -2 \\
0 & 0 & 0 & 0 & 0 \\
2 & 8 & 12 & 8 & 2 \\
1 & 4 & 6 & 4 & 1 \\
\end{bmatrix}$$

- **垂直邊緣檢測 ($7\times 7$)**：

$$
\begin{bmatrix}
-1 & -4 & -5 & -6 & -5 & -4 & -1 \\
-2 & -8 & -15 & -24 & -15 & -8 & -2 \\
-3 & -12 & -20 & -30 & -20 & -12 & -3 \\
0 & 0 & 0 & 0 & 0 & 0 & 0 \\
3 & 12 & 20 & 30 & 20 & 12 & 3 \\
2 & 8 & 15 & 24 & 15 & 8 & 2 \\
1 & 4 & 5 & 6 & 5 & 4 & 1 \\
\end{bmatrix}$$

- 使用 $7 \times 7$ 的卷積核，模型可以在更大的空間範圍內檢測邊緣，適合於圖像尺寸較大、細節重要性低於結構的應用。

---

#### 1.2. **Prewitt 運算子**

- Prewitt 運算子與 Sobel 相似，但採用了更簡單的平均方法，因此計算成本較低。
- Prewitt 運算子能夠檢測與 Sobel 類似的邊緣，但其計算成本較低。

- **水平邊緣檢測**：

$$
\begin{bmatrix}
-1 & 0 & 1 \\
-1 & 0 & 1 \\
-1 & 0 & 1 \\
\end{bmatrix}$$

- **垂直邊緣檢測**：

$$
\begin{bmatrix}
-1 & -1 & -1 \\
0 & 0 & 0 \\
1 & 1 & 1 \\
\end{bmatrix}$$

---

#### 1.3. **Scharr 運算子**

- Scharr 運算子是 Sobel 運算子的改進版，對中心像素賦予更大的權重，以提高檢測的精度。
- Scharr 濾波器比 Sobel 和 Prewitt 更敏感，通常能夠提供更精細的檢測結果。

- **水平邊緣檢測 ($3\times 3$)**：

$$
\begin{bmatrix}
3 & 0 & -3 \\
10 & 0 & -10 \\
3 & 0 & -3 \\
\end{bmatrix}$$

- **垂直邊緣檢測 ($3\times 3$)**：

$$
\begin{bmatrix}
3 & 10 & 3 \\
0 & 0 & 0 \\
-3 & -10 & -3 \\
\end{bmatrix}$$

---

#### 1.4. **拉普拉斯運算子（Laplacian Operator）**

- 拉普拉斯運算子通過檢測強度變化來檢測邊緣，通常將水平和垂直邊緣檢測結合在一個卷積核中。
- 拉普拉斯卷積核能夠檢測不同方向的邊緣，適合強調所有方向的強度變化。

- **4-連通的拉普拉斯卷積核 ($3\times 3$)**：

$$
\begin{bmatrix}
0 & -1 & 0 \\
-1 & 4 & -1 \\
0 & -1 & 0 \\
\end{bmatrix}$$

- **8-連通的拉普拉斯卷積核 ($3\times 3$)**：

$$
\begin{bmatrix}
-1 & -1 & -1 \\
-1 & 8 & -1 \\
-1 & -1 & -1 \\
\end{bmatrix}$$

- **拉普拉斯卷積核 ($5\times 5$)**：

$$
\begin{bmatrix}
0 & 0 & -1 & 0 & 0 \\
0 & -1 & -2 & -1 & 0 \\
-1 & -2 & 16 & -2 & -1 \\
0 & -1 & -2 & -1 & 0 \\
0 & 0 & -1 & 0 & 0 \\
\end{bmatrix}$$

- 中心的權重（此處為 16）比 $3 \times 3$ 的拉普拉斯高，這有助於放大較大區域的變化，用來檢測較大的邊緣或漸變。

- **拉普拉斯卷積核 ($7\times 7$)**：

$$
\begin{bmatrix}
0 & 0 & 0 & -1 & 0 & 0 & 0 \\
0 & 0 & -1 & -2 & -1 & 0 & 0 \\
0 & -1 & -2 & -4 & -2 & -1 & 0 \\
-1 & -2 & -4 & 20 & -4 & -2 & -1 \\
0 & -1 & -2 & -4 & -2 & -1 & 0 \\
0 & 0 & -1 & -2 & -1 & 0 & 0 \\
0 & 0 & 0 & -1 & 0 & 0 & 0 \\
\end{bmatrix}$$

- 高中心值（20）確保濾波器對較大的強度變化有強烈的響應，適合於圖像中的大結構檢測。

---

#### 1.5. **Roberts 交叉運算子（Roberts Cross Operator）**

- Roberts 交叉運算子使用 2x2 卷積核來檢測邊緣，通常用於快速檢測對角線邊緣。
- 這些小型卷積核對對角邊緣特別敏感，能夠快速檢測小的強度變化。

- **對角線邊緣檢測 1 ($3\times 3$)**：

$$
\begin{bmatrix}
1 & 0 \\
0 & -1 \\
\end{bmatrix}$$

- **對角線邊緣檢測 2 ($3\times 3$)**：

$$
\begin{bmatrix}
0 & 1 \\
-1 & 0 \\
\end{bmatrix}$$

---

#### 1.6. **Kirsch 指南針運算子（Kirsch Compass Operator）**

- Kirsch 運算子使用一組八個卷積核，每個卷積核檢測不同的方向（北、東北、東、東南、南、西南、西、西北）。
- 每個方向都有一個獨特的卷積核，使得 Kirsch 運算子能夠有效檢測任何方向的邊緣。

- **北方向**：

$$
\begin{bmatrix}
5 & 5 & 5 \\
-3 & 0 & -3 \\
-3 & -3 & -3 \\
\end{bmatrix}$$

- **東方向**：

$$
\begin{bmatrix}
-3 & -3 & 5 \\
-3 & 0 & 5 \\
-3 & -3 & 5 \\
\end{bmatrix}$$

---

#### 1.7 Canny 邊緣檢測的步驟

1. **降噪**：首先對圖像應用高斯模糊來減少噪聲，使邊緣檢測更為準確。
2. **梯度計算**：接著使用 Sobel 濾波器計算圖像中的亮度梯度，這樣可以識別出潛在的邊緣區域。
3. **非極大值抑制**：對邊緣進行細化，只保留在梯度方向上具有最大值的像素點，從而去除不必要的邊緣。
4. **雙閾值處理**：使用高、低閾值將邊緣分為強邊緣和弱邊緣，進行分類。
5. **滯後邊緣跟踪**：從強邊緣開始，將與之相連的弱邊緣也標記為邊緣，其餘像素則被丟棄。

##### Canny 邊緣檢測使用的卷積核

Canny 算法主要依賴於高斯平滑和 Sobel 運算子，以下是每一步中常用的卷積核。

##### 高斯模糊卷積核

在平滑過程中，經常使用不同尺寸的高斯卷積核。以下是 $3 \times 3$, $5 \times 5$ 和 $7 \times 7$ 的高斯卷積核示例。

1. **$3\times3$ 高斯卷積核**：

$$
\frac{1}{16} \begin{bmatrix}
1 & 2 & 1 \\
2 & 4 & 2 \\
1 & 2 & 1 \\
\end{bmatrix}$$

2. **$5\times5$ 高斯卷積核**：

$$
\frac{1}{273} \begin{bmatrix}
1 & 4 & 7 & 4 & 1 \\
4 & 16 & 26 & 16 & 4 \\
7 & 26 & 41 & 26 & 7 \\
4 & 16 & 26 & 16 & 4 \\
1 & 4 & 7 & 4 & 1 \\
\end{bmatrix}$$

3. **$7\times7$ 高斯卷積核**：

$$
\frac{1}{1003} \begin{bmatrix}
0 & 0 & 3 & 5 & 3 & 0 & 0 \\
0 & 5 & 11 & 16 & 11 & 5 & 0 \\
3 & 11 & 21 & 26 & 21 & 11 & 3 \\
5 & 16 & 26 & 33 & 26 & 16 & 5 \\
3 & 11 & 21 & 26 & 21 & 11 & 3 \\
0 & 5 & 11 & 16 & 11 & 5 & 0 \\
0 & 0 & 3 & 5 & 3 & 0 & 0 \\
\end{bmatrix}$$

- 這些高斯卷積核在降噪步驟中用於平滑圖像，減少噪聲對邊緣檢測的影響。

##### Sobel 梯度卷積核

在平滑後，應用 Sobel 運算子來檢測圖像中的 x 和 y 方向上的梯度，通常使用 $3 \times 3$ 的卷積核，但在特定需求下也可以擴展為更大的尺寸。

1. **$3\times3$ Sobel X 方向梯度卷積核**：

$$
\begin{bmatrix}
-1 & 0 & 1 \\
-2 & 0 & 2 \\
-1 & 0 & 1 \\
\end{bmatrix}$$

2. **$3\times3$ Sobel Y 方向梯度卷積核**：

$$
\begin{bmatrix}
-1 & -2 & -1 \\
0 & 0 & 0 \\
1 & 2 & 1 \\
\end{bmatrix}$$

- 在 Canny 邊緣檢測中，這些 Sobel 濾波器被用來計算圖像的強度梯度，以幫助識別亮度變化快速的邊緣。

#### 小結

- 這些卷積核的設計針對不同的邊緣方向或強度變化，使得它們可以根據應用的具體需求選擇適合的邊緣檢測工具。Sobel、Prewitt 和 Scharr 常用於一般邊緣檢測，而 Laplacian、Roberts 和 Kirsch 更適合用於特定的邊緣檢測任務。
- **$5\times5$ 卷積核** 通常用於需要更細節但略大區域的檢測，比 $3 \times 3$ 卷積核範圍更大。
- **$7\times7$ 卷積核** 適合於檢測更廣泛的圖案，特別是高解析度圖像，當邊緣的重要性大於細節時，這些較大的卷積核非常實用。
- 這些較大的卷積核可以捕捉更多的信息，適合於 $3 \times 3$ 卷積核可能忽略的大範圍或較不明顯的邊緣。但它們也具有更高的計算成本，因此通常僅在 CNN 架構的特定層中選擇性地使用。
- **高斯卷積核**（ $3\times3$、 $5\times5$、 $7\times7$）：用於圖像平滑處理。
- **Sobel 卷積核**（通常為 $3\times3$）：用於計算 x 和 y 方向的梯度，從而檢測出邊緣。

---

### 2. **銳化卷積核**

- 銳化卷積核強調邊緣和細節，使圖像更加清晰。
- 這個卷積核會放大中心像素與其周圍像素之間的差異，從而增強邊緣和細節。

$$
\begin{bmatrix}
0 & -1 & 0 \\
-1 & 5 & -1 \\
0 & -1 & 0 \\
\end{bmatrix}$$

---

### 3. **模糊卷積核（方框模糊）**

- 模糊卷積核會對相鄰像素進行平均，從而使圖像變得平滑，這在降噪或進行特徵提取之前的預處理中非常有用。
- 高斯模糊使用加權平均，比方框模糊保留更多細節。

- **方框模糊 ($3\times3$)**：

$$
\frac{1}{9} \begin{bmatrix}
1 & 1 & 1 \\
1 & 1 & 1 \\
1 & 1 & 1 \\
\end{bmatrix}$$

- **高斯模糊 ($3\times3$)**：

$$
\frac{1}{16} \begin{bmatrix}
1 & 2 & 1 \\
2 & 4 & 2 \\
1 & 2 & 1 \\
\end{bmatrix}$$

---

### 4. **浮雕卷積核**

- 浮雕卷積核可以創造出3D效果，強調圖像的紋理和細節。
- 這種卷積核可以產生浮雕效果，一側有光亮，另一側有陰影，非常適合強調圖像的質感。

$$
\begin{bmatrix}
-2 & -1 & 0 \\
-1 & 1 & 1 \\
0 & 1 & 2 \\
\end{bmatrix}$$

---

### 5. **單位卷積核**

- 單位卷積核保留了原始圖像的特徵，可以用作基準，或幫助理解其他卷積核的行為。

$$
\begin{bmatrix}
0 & 0 & 0 \\
0 & 1 & 0 \\
0 & 0 & 0 \\
\end{bmatrix}$$

---

### 6. **自定義特徵提取卷積核**

- 在實踐中，CNN 通過訓練學習到的自定義卷積核可能較難以視覺化，但通常會捕捉到更抽象的特徵，如角點、紋理或特定形狀。
