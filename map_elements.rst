======================================
加入地圖元素
======================================
為了使呈現的資料更容易判讀，一張地圖上通常會含有其它的輔助性圖示。除了網格線和地圖四周的座標數字外，常見的圖示還有\
比例尺、指北記號以及分圖等等。在本章中，我們要來試著使用 GMT 標上這些地圖元素。

目標
--------------------------------------
繪製\ `魯卜哈利沙漠 <https://zh.wikipedia.org/wiki/%E9%B2%81%E5%8D%9C%E5%93%88%E5%88%A9%E6%B2%99%E6%BC%A0>`_
(Rub' al Khali 或 الربع الخالي) 其中一個\
小區域的 `Landsat <https://zh.wikipedia.org/wiki/%E9%99%B8%E5%9C%B0%E8%A1%9B%E6%98%9F%E8%A8%88%E7%95%AB>`_
衛星影像，並標上指北針、比例尺及在阿拉伯半島的大略位置。魯卜哈利沙漠具有全世界最大的連續沙丘區域\ [#]_\ ，也是全世界最杳無人煙的地方之一。\
如同其阿拉伯語的意思「空曠的四分之一」，在這片區域除了沙丘以外幾乎什麼也沒有；\
然而，魯卜哈利沙漠的衛星影像卻讓我們發現了大地之母的藝術實驗室。無數的\
`新月丘 <https://en.wikipedia.org/wiki/Barchan>`_\ 連綿不絕，日以繼夜的移動、合併與分離，\
奇幻的畫面甚至會讓你懷疑這是否是真的自然景觀。由於以上特殊的地理、氣候條件，魯卜哈利沙漠也是研究由風力所形塑的地表作用的絕佳地點。\
在本衛星影像中，所有的新月形沙丘似乎都朝向相同的方向。這個事實告訴了我們在這個區域，盛行風是從西北偏北的方向吹來。\
為什麼？請看\ `這裡 <http://nadm.gl.ntu.edu.tw/nadm/cht/class_detail.php?serial=122&serial_type_1=8&serial_type_2=4&serial_type_3=16>`_\ 。

.. _最終版地圖:

.. image:: map_elements/Rub_al_Khali_s.png
    :target: _images/Rub_al_Khali.png

直接觀看\ `指令稿`_


使用的指令與概念
--------------------------------------
- ``gmtset`` - 更改 GMT 的預設作圖參數
- ``grdhisteq`` - **查看網格檔的數值分佈**
- ``grdinfo`` - **由網格資料建立** ``-R`` **設定**
- ``grdimage`` - **以 RGB 資料進行繪圖**
- ``psbasemap`` - **添加不同的地圖元素**
- ``pscoast`` - 繪製海岸線、國界與海域填色
- ``pstext`` - 在地圖上加入文字
- ``psxy`` - 輸出檔尾
- ``psconvert`` - 把 ps 檔轉成 png 檔並保留透明度設定
- 外部指令 ``echo`` - 把資料輸入到管線命令中
- 如何使用正確的 ``-J`` 投影選項套疊衛星影像與其他地理空間資料
- 如何繪製分圖 (Map Inset)

前置作業
--------------------------------------
Landsat 衛星影像可以從美國地質調查局 (USGS) 管理的網站 `EarthExplorer <https://earthexplorer.usgs.gov/>`_ 取得。\
前往 EarthExplorer，然後在「Address/Place」搜尋的地方輸入 *Rub’ al Khali*，然後按下 `show`，選擇下方出現\
的 *魯卜哈利沙漠* 一欄。由於沙漠範圍很大，你可以拖曳地圖上的紅色定位點到較為靠近範例地圖的地方。接著，按下下方的 `Data Sets >>`。\
你可以使用搜尋框或下方的分類目錄，找到並勾選 *Landsat 8 OLI/TIRS C1 Level-1* 的資料集，然後按下 `Results >>`。

.. image:: map_elements/map_elements_fig1.png

接下來，應該會有許多資料呈現在左側搜尋結果欄中。找到名為 *LC08_L1TP_160045_20170408_20170414_01_T1* 的資料，\
按下「Download Options」的圖示。本資料需要登入才能下載，如果你尚未有 USGS 帳號，可以花幾分鐘註冊，\
只需要一組電子信箱即可。登入之後，選擇 `Level-1 GeoTIFF Data Product (886.0 MB)` 的資料格式。

.. image:: map_elements/map_elements_fig2.png

下載之後解壓縮，你會發現其中有許多 GeoTIFF 檔案，每個檔案代表一個特定的影像波段。\
`Landsat 網站上 <https://landsat.usgs.gov/what-are-band-designations-landsat-satellites>`_\ 說明了每個編號各自代表的波段。在本教學中，\
我們要使用可見光的紅、綠、藍三個波段 (RGB)，編號分別是 B4、B3 和 B2。為了方便分享輸入檔以及增強影像的對比，我對這三個影像進行了\
如下操作：

1. 裁切影像，影像的邊緣座標定在 x = [190000 250000] 以及 y = [2410000 2460000] (UTM 座標)。
2. 使用直方圖增強影像對比 (Histogram Stretching)。
3. 轉檔成 NetCDF 格式。

使用的腳本如下所示，其中運用了 GDAL 函式庫中的 ``gdal_translate`` 跟 GMT 中的 ``grdhisteq`` 指令。\
相關的操作將在\ **之後的章節**\ 中介紹，這裡就不多加贅述，把重心放在地圖元素的繪製上。

.. code-block:: bash

    # ==== 設定輸入檔 ====
    in_tif_r="LC08_L1TP_160045_20170408_20170414_01_T1_B4.TIF"
    in_tif_g="LC08_L1TP_160045_20170408_20170414_01_T1_B3.TIF"
    in_tif_b="LC08_L1TP_160045_20170408_20170414_01_T1_B2.TIF"
    # ==== 使用迴圈逐一處理輸入檔 ====
    for i in $in_tif_r $in_tif_g $in_tif_b; do
        # ==== 裁切 GeoTIFF 影像 ====
        gdal_translate -projwin 190000 2460000 250000 2410000 $i ${i/.TIF/_s.TIF}
        # ==== 計算影像的直方圖，設定影像飽和 (黑色或白色) 時的像素值 ====
        vmin=$(grdhisteq ${i/.TIF/_s.TIF} -D -C50 | head -n 1 | cut -f 2)
        vmax=$(grdhisteq ${i/.TIF/_s.TIF} -D -C50 | tail -n 1 | cut -f 1)
        # ==== 使用剛剛設定的影像飽和值來轉檔 ====
        gdal_translate -of NetCDF -scale $vmin $vmax ${i/.TIF/_s.TIF} ${i/.TIF/_s.grd}
    done

腳本的輸出是三個 ``.grd`` 檔。如果你的電腦沒有安裝 GDAL 或純粹為了方便起見，
你也可以直接從以下連結取得本章節會使用的 Landsat 8 ``.grd`` 檔案：

:download:`LC08_L1TP_160045_20170408_20170414_01_T1_B2_s.grd <map_elements/LC08_L1TP_160045_20170408_20170414_01_T1_B2_s.grd>`
:download:`LC08_L1TP_160045_20170408_20170414_01_T1_B3_s.grd <map_elements/LC08_L1TP_160045_20170408_20170414_01_T1_B3_s.grd>`
:download:`LC08_L1TP_160045_20170408_20170414_01_T1_B4_s.grd <map_elements/LC08_L1TP_160045_20170408_20170414_01_T1_B4_s.grd>`

操作流程
--------------------------------------

在繪圖之前，我們先來使用 ``grdinfo`` 確認一下要使用的資料。

.. code-block:: bash

    $ grdinfo LC08_L1TP_160045_20170408_20170414_01_T1_B2_s.grd
    ..... #(我這裡略過了一些段落)
    LC08_L1TP_160045_20170408_20170414_01_T1_B2_s.grd: Title: GDAL Band Number 1
    LC08_L1TP_160045_20170408_20170414_01_T1_B2_s.grd: Grid file format: nf = GMT netCDF format (32-bit float), COARDS, CF-1.5
    LC08_L1TP_160045_20170408_20170414_01_T1_B2_s.grd: x_min: 190015 x_max: 249985 x_inc: 30 name: x coordinate of projection [m] nx: 2000
    LC08_L1TP_160045_20170408_20170414_01_T1_B2_s.grd: y_min: 2410014.997 y_max: 2459985.003 y_inc: 29.9940011998 name: y coordinate of projection [m] ny: 1667
    LC08_L1TP_160045_20170408_20170414_01_T1_B2_s.grd: z_min: NaN z_max: NaN name: GDAL Band Number 1

從這裡可以看出我們的資料並不是使用經緯度座標，而是某個投影座標，每個像素的尺寸是 30 公尺。如果使用 ``gdalinfo`` 確認原本的
GeoTIFF 檔，可以進一步知道這個座標系統是 UTM Zone 40N。像素的最大與最小值在這裡錯誤的顯示成 NaN，我們需要使用 ``grdhisteq`` 來進一步的檢查。\
``grdhisteq`` 是 GMT 中對網格檔進行數值統計的工具，我們要使用最基本的語法，在螢幕上輸出網格檔數值的分佈情形。

.. code-block:: bash

    $ grdhisteq -D LC08_L1TP_160045_20170408_20170414_01_T1_B2_s.grd
    # -D: 輸出資訊到檔案或螢幕上
    0	14	0
    14	25	1
    25	35	2
    35	42	3
    42	49	4
    49	56	5
    56	63	6
    63	71	7
    71	79	8
    79	91	9
    91	111	10
    111	139	11
    139	167	12
    167	192	13
    192	221	14
    221	587	15

``grdhisteq`` 預設的輸出，是把所有的像素值依照大小均分成 16 等分。從輸出資料中，我們可以知道像素值位在 0 到 14 之間的像素\
佔了十六分之一，而像素值在 221 到 587 之間的像素也佔了十六分之一。接下來，我們就試著使用 ``grdimage`` 來輸出影像看看。\
在之前的章節，我們使用 ``grdimage`` 時需要給定對應的色階檔，讓指令照著輸入檔像素值的大小來給定顏色。如果我們想要不經由色階檔\
直接指定每個像素的顏色，``grdimage`` 也是可以作到，不過要遵循以下的輸入方法：























指令稿
--------------------------------------
本地圖的最終指令稿如下：

.. code-block:: bash

    # ==== 設定變數 ====
    out_ps="Rub_al_Khali.ps"
    in_grd_r="LC08_L1TP_160045_20170408_20170414_01_T1_B4_s.grd"    # 輸入檔，作為紅色 (R) 波段
    in_grd_g="LC08_L1TP_160045_20170408_20170414_01_T1_B3_s.grd"    # 輸入檔，作為綠色 (G) 波段
    in_grd_b="LC08_L1TP_160045_20170408_20170414_01_T1_B2_s.grd"    # 輸入檔，作為藍色 (B) 波段

    # ==== 調整 GMT 預設參數 ====
    gmtset MAP_FRAME_TYPE=plain

    # ==== 繪製衛星影像 ====
        # 取得 -R 資訊並畫圖 (注意 -J 的設定！)
    R=$(grdinfo $in_grd_r -I1/1)
    grdimage $in_grd_r $in_grd_g $in_grd_b -R$in_grd_r -JX15c/0 -P -K > $out_ps
        # 繪製不含任何標記的邊框
    psbasemap -R -J -O -K -B0 >> $out_ps

    # ==== 繪製地圖元素 ====
        # 繪製指北針 (注意 -J 的設定！)
            # GMT 5.2
    # psbasemap -R -JU40Q/15c -O -K -TdjRT+w2c+f+l\ ,\ ,\ ,N+o1c/1.8c -F+c0.2c/0.2c/0.2c/1c+gwhite@50+r0.5c --FONT=15p >> $out_ps
            # GMT 5.4
    psbasemap -R -JU40Q/15c -O -K -TdjRT+w2c+f+l,,,N+o1c/1.8c -F+c0.2c/0.2c/0.2c/1c+gwhite@50+r0.5c --FONT=15p >> $out_ps
        # 繪製比例尺
    psbasemap -R -J         -O -K -LjRB+c22+w10k+f+o2c/2c+u -F+gwhite@50  >> $out_ps
        # 繪製分圖，放上鄰近區域的海岸線和國界位置
    pscoast -R-700000/1100000/1920000/3180000 -JU40Q/5c -O -K -Di -Wthinner -Gwhite -Sgray -B0 -N1/thinnest >> $out_ps
        # 標上主地圖在分圖中的位置 (注意 -J 的設定！) 與其他文字
    psbasemap -R -JX5c/0 -O -K -D${R#*-R} -F >> $out_ps
    echo -300000 2300000 Saudi Arabia | pstext -R -J -O -K -F+f8p+jMC >> $out_ps

    # ==== 關門 (寫入 EOF) 與轉檔 ====
    psxy -R -J -O -T >> $out_ps
    psconvert $out_ps -A -P -Tg

.. note::

    「」

觀看\ `最終版地圖`_

習題
--------------------------------------


.. [#] Peter Vincent (2008). 
       `Saudi Arabia: an environmental overview <https://books.google.com/books?id=Vacv2wy3yd8C&pg=PA141>`_. 
       Taylor & Francis. p. 141. ISBN 978-0-415-41387-9.

