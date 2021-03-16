# 边缘检测

> 利用边缘检测对HSI的相似矩阵进行分组，可以使得后面操作变成在分组中使用CCE方法

## 1、基本操作步骤

灰度处理->加噪声->去除噪声->使用不同的算子对image进行卷积

## 2、Matlab代码

以Idiana Pines为例

```matlab
%RGB_YCbCr
clc;
clear all;
close all;

% RGB_data = imread('similarityMatrix.jpg');%相似矩阵
RGB_data = imread('sim185.png');%normalize 后的相似矩阵

R_data =    RGB_data(:,:,1);
G_data =    RGB_data(:,:,2);
B_data =    RGB_data(:,:,3);

%imshow(RGB_data);

[ROW,COL, DIM] = size(RGB_data);

Y_data = zeros(ROW,COL);
Cb_data = zeros(ROW,COL);
Cr_data = zeros(ROW,COL);
Gray_data = RGB_data;

for r = 1:ROW
    for c = 1:COL
        Y_data(r, c) = 0.299*R_data(r, c) + 0.587*G_data(r, c) + 0.114*B_data(r, c);
        Cb_data(r, c) = -0.172*R_data(r, c) - 0.339*G_data(r, c) + 0.511*B_data(r, c) + 128;
        Cr_data(r, c) = 0.511*R_data(r, c) - 0.428*G_data(r, c) - 0.083*B_data(r, c) + 128;
    end
end

Gray_data(:,:,1)=Y_data;
Gray_data(:,:,2)=Y_data;
Gray_data(:,:,3)=Y_data;

% figure;
% imshow(Gray_data);

%Median Filter      中值滤波
imgn = imnoise(Gray_data,'salt & pepper',0.02); %add noise to image

% figure;
% imshow(imgn);

Median_Img = Gray_data;
for r = 2:ROW-1
    for c = 2:COL-1
        median3x3 =[imgn(r-1,c-1)    imgn(r-1,c) imgn(r-1,c+1)
                    imgn(r,c-1)      imgn(r,c)      imgn(r,c+1)
                    imgn(r+1,c-1)      imgn(r+1,c) imgn(r+1,c+1)];
        sort1 = sort(median3x3, 2, 'descend');
        sort2 = sort([sort1(1), sort1(4), sort1(7)], 'descend');
        sort3 = sort([sort1(2), sort1(5), sort1(8)], 'descend');
        sort4 = sort([sort1(3), sort1(6), sort1(9)], 'descend');
        mid_num = sort([sort2(3), sort3(2), sort4(1)], 'descend');
        Median_Img(r,c) = mid_num(2);
    end
end

% figure;
% imshow(Median_Img);

%Gaussian Filter
%parameter: sigma
sigma = 3;
GaussianModel = getGaussianMat(sigma);
for r = 2:ROW-1
    for c = 2:COL-1
        median3x3 =[Median_Img(r-1,c-1)      Median_Img(r-1,c)      Median_Img(r-1,c+1)
                    Median_Img(r,c-1)        Median_Img(r,c)        Median_Img(r,c+1)
                    Median_Img(r+1,c-1)      Median_Img(r+1,c)      Median_Img(r+1,c+1)];
        median3x3 = double(median3x3);
        filterNum = floor(sum(sum(median3x3.*GaussianModel,2)));
        Median_Img(r,c) = filterNum;
    end
end
figure;
imshow(Median_Img);


% %Sobel_Edge_Detect
% 
% Median_Img = double(Median_Img);
% Sobel_Threshold = 55;
% Sobel_Img = zeros(ROW,COL);
% for r = 2:ROW-1
%     for c = 2:COL-1
%         Sobel_x = Median_Img(r-1,c+1) + 2*Median_Img(r,c+1) + Median_Img(r+1,c+1) - Median_Img(r-1,c-1) - 2*Median_Img(r,c-1) - Median_Img(r+1,c-1);
%         Sobel_y = Median_Img(r-1,c-1) + 2*Median_Img(r-1,c) + Median_Img(r-1,c+1) - Median_Img(r+1,c-1) - 2*Median_Img(r+1,c) - Median_Img(r+1,c+1);
%         Sobel_Num = abs(Sobel_x) + abs(Sobel_y);
%         %Sobel_Num = sqrt(Sobel_x^2 + Sobel_y^2);
%         if(Sobel_Num > Sobel_Threshold)
%             Sobel_Img(r,c)=0;
%         else
%             Sobel_Img(r,c)=255;
%         end
%     end
% end
% 
% figure;
% imshow(Sobel_Img);

% %Robert_Edge_Detect
% 
% Median_Img = double(Median_Img);
% Robert_Threshold = 15;
% Rober_Img = zeros(ROW,COL);
% for r = 2:ROW-1
%     for c = 2:COL-1
%         Robert_x = Median_Img(r+1,c+1) - Median_Img(r,c);
%         Robert_y = Median_Img(r+1,c) - Median_Img(r,c+1);
%         Robert_Num = abs(Robert_x) + abs(Robert_y);
%         %Robert_Num = sqrt(Sobel_x^2 + Sobel_y^2);
%         if(Robert_Num > Robert_Threshold)
%             Rober_Img(r,c)=0;
%         else
%             Rober_Img(r,c)=255;
%         end
%     end
% end
% 
% figure;
% imshow(Rober_Img);

%Prewitt_Edge_Detect

Median_Img = double(Median_Img);
Prewitt_Threshold = 20;
Prewitt_Img = zeros(ROW,COL);
for r = 2:ROW-1
    for c = 2:COL-1
        Prewitt_x = (Median_Img(r+1,c-1)+Median_Img(r+1,c)+Median_Img(r+1,c+1))-(Median_Img(r-1,c-1)+Median_Img(r-1,c)+Median_Img(r-1,c+1));
        Prewitt_y = (Median_Img(r-1,c+1)+Median_Img(r,c+1)+Median_Img(r+1,c+1))-(Median_Img(r-1,c-1)+Median_Img(r,c-1)+Median_Img(r+1,c-1));
        Prewitt_Num = abs(Prewitt_x) + abs(Prewitt_y);
        %Robert_Num = sqrt(Sobel_x^2 + Sobel_y^2);
        if(Prewitt_Num > Prewitt_Threshold)
            Prewitt_Img(r,c)=0;
        else
            Prewitt_Img(r,c)=255;
        end
    end
end

% %resize the picture
% Temp_Prewitt_Img = imresize(Prewitt_Img, [185,185]);
% figure;
% imshow(Temp_Prewitt_Img);
% axis on;
% impixelinfo;

figure;
imshow(Prewitt_Img);
axis on;
impixelinfo;
```

## 3、调整的参数

在边缘探测中，我们使用了高斯模糊以及相应的边缘检测算法

算子：

- sobel
- robert
- prewitt

高斯模糊参数

- $ \sigma $

边缘检测参数

- Threshold （越大分隔得越少块）



## 4、example

参数

- prewitt算子
- $\sigma$ = 3
- Threshold = 35

![img](https://gblobscdn.gitbook.com/assets%2F-MVu4pfg1FQYKC-Y6Jr9%2F-MVu7NxBZECa9CRx5B_I%2F-MVuDTZ65qzSfBcp2yr3%2Fimage.png?alt=media&token=ce120635-f4af-4389-bc47-c411c62204d2)

![img](https://gblobscdn.gitbook.com/assets%2F-MVu4pfg1FQYKC-Y6Jr9%2F-MVu7NxBZECa9CRx5B_I%2F-MVuDfpQGFmw27h6bDhl%2Fimage.png?alt=media&token=4ae45624-0b33-471e-b7ee-e4b72727a892)



## 5、相应参数锁对应的分组

[实验数据](Data/Edge_Detect_data.xls)





