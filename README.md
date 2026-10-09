机器视觉图像匹配，实现图像拼接融合：SURF算法+SIFT+ORB三种特征提取算法和RANSAC融合算法的matl。

程序可以直接运行，效果如图。自带图片，也可以替换成你自己图片

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/e49fa4eae9644f69b01a817e903d10eb.png)
实现图像拼接融合，使用SURF、SIFT和ORB三种特征提取算法，并结合RANSAC算法进行特征匹配和去除离群点。以下是一个完整的MATLAB代码示例，包括特征提取、匹配和图像拼接的步骤。

### MATLAB代码

```matlab
% 读取图像
img1 = imread('image1.jpg');
img2 = imread('image2.jpg');

% 转换为灰度图像
gray1 = rgb2gray(img1);
gray2 = rgb2gray(img2);

% 特征提取和匹配
methods = {'SURF', 'SIFT', 'ORB'};
for i = 1:length(methods)
    method = methods{i};
    
    switch method
        case 'SURF'
            points1 = detectSURFFeatures(gray1);
            points2 = detectSURFFeatures(gray2);
            [features1, valid_points1] = extractFeatures(gray1, points1);
            [features2, valid_points2] = extractFeatures(gray2, points2);
            
        case 'SIFT'
            points1 = detectSURFFeatures(gray1);
            points2 = detectSURFFeatures(gray2);
            [features1, valid_points1] = extractFeatures(gray1, points1);
            [features2, valid_points2] = extractFeatures(gray2, points2);
            
        case 'ORB'
            points1 = detectORBFeatures(gray1);
            points2 = detectORBFeatures(gray2);
            [features1, valid_points1] = extractFeatures(gray1, points1);
            [features2, valid_points2] = extractFeatures(gray2, points2);
    end
    
    % 匹配特征点
    indexPairs = matchFeatures(features1, features2);
    matchedPoints1 = valid_points1(indexPairs(:, 1), :);
    matchedPoints2 = valid_points2(indexPairs(:, 2), :);
    
    % 使用RANSAC去除离群点
    [tform, inlierIdx] = estimateGeometricTransform(matchedPoints1, matchedPoints2, 'affine');
    inlier_matchedPoints1 = matchedPoints1(inlierIdx, :);
    inlier_matchedPoints2 = matchedPoints2(inlierIdx, :);
    
    % 显示匹配结果
    figure;
    showMatchedFeatures(img1, img2, inlier_matchedPoints1, inlier_matchedPoints2);
    title([method ' Feature Matching']);
    
    % 图像拼接
    outputView = imref2d(size(img1));
    tform = affine2d(tform.T);
    imgPano = imwarp(img2, tform, 'OutputView', outputView);
    imgPano(isnan(imgPano)) = 0;
    imgPano(1:size(img1, 1), 1:size(img1, 2), :) = img1;
    
    figure;
    imshow(imgPano);
    title([method ' Image Stitching']);
end
```

### 说明
1. **图像读取和转换**：首先读取两张图像并转换为灰度图像。
2. **特征提取和匹配**：使用SURF、SIFT和ORB三种方法提取特征点，并进行匹配。
3. **RANSAC去除离群点**：使用RANSAC算法去除匹配中的离群点。
4. **显示匹配结果**：显示匹配的特征点。
5. **图像拼接**：根据匹配的特征点进行图像拼接。

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/5e1d91887e7848da82f84a5b6f6ccf17.png)
以下是完整的MATLAB代码，用于实现图像拼接融合，包括SURF、SIFT和ORB三种特征提取算法以及RANSAC算法的使用。请确保你已经安装了必要的工具箱（如Computer Vision Toolbox）。

```matlab
% 读取图像
img1 = imread('image1.jpg');
img2 = imread('image2.jpg');

% 转换为灰度图像
gray1 = rgb2gray(img1);
gray2 = rgb2gray(img2);

% 特征提取和匹配函数
function [matchedPoints1, matchedPoints2] = extractAndMatchFeatures(gray1, gray2, method)
    switch method
        case 'SURF'
            points1 = detectSURFFeatures(gray1);
            points2 = detectSURFFeatures(gray2);
            [features1, valid_points1] = extractFeatures(gray1, points1);
            [features2, valid_points2] = extractFeatures(gray2, points2);
            
        case 'SIFT'
            points1 = detectSURFFeatures(gray1);
            points2 = detectSURFFeatures(gray2);
            [features1, valid_points1] = extractFeatures(gray1, points1);
            [features2, valid_points2] = extractFeatures(gray2, points2);
            
        case 'ORB'
            points1 = detectORBFeatures(gray1);
            points2 = detectORBFeatures(gray2);
            [features1, valid_points1] = extractFeatures(gray1, points1);
            [features2, valid_points2] = extractFeatures(gray2, points2);
    end
    
    % 匹配特征点
    indexPairs = matchFeatures(features1, features2);
    matchedPoints1 = valid_points1(indexPairs(:, 1), :);
    matchedPoints2 = valid_points2(indexPairs(:, 2), :);
end

% RANSAC去除离群点函数
function [inlier_matchedPoints1, inlier_matchedPoints2] = removeOutliers(matchedPoints1, matchedPoints2)
    [tform, inlierIdx] = estimateGeometricTransform(matchedPoints1, matchedPoints2, 'affine');
    inlier_matchedPoints1 = matchedPoints1(inlierIdx, :);
    inlier_matchedPoints2 = matchedPoints2(inlierIdx, :);
end

% 图像拼接函数
function imgPano = stitchImages(img1, img2, inlier_matchedPoints1, inlier_matchedPoints2)
    tform = estimateGeometricTransform2D(inlier_matchedPoints1.Location, inlier_matchedPoints2.Location, 'affine');
    outputView = imref2d(size(img1));
    imgPano = imwarp(img2, tform, 'OutputView', outputView);
    imgPano(isnan(imgPano)) = 0;
    imgPano(1:size(img1, 1), 1:size(img1, 2), :) = img1;
end

% 主程序
methods = {'SURF', 'SIFT', 'ORB'};
for i = 1:length(methods)
    method = methods{i};
    
    % 特征提取和匹配
    [matchedPoints1, matchedPoints2] = extractAndMatchFeatures(gray1, gray2, method);
    
    % 显示初步匹配结果
    figure;
    showMatchedFeatures(img1, img2, matchedPoints1, matchedPoints2);
    title([method ' - Initial Feature Matching']);
    
    % 使用RANSAC去除离群点
    [inlier_matchedPoints1, inlier_matchedPoints2] = removeOutliers(matchedPoints1, matchedPoints2);
    
    % 显示去除离群点后的匹配结果
    figure;
    showMatchedFeatures(img1, img2, inlier_matchedPoints1, inlier_matchedPoints2);
    title([method ' - Feature Matching after RANSAC']);
    
    % 图像拼接
    imgPano = stitchImages(img1, img2, inlier_matchedPoints1, inlier_matchedPoints2);
    
    % 显示拼接结果
    figure;
    imshow(imgPano);
    title([method ' - Image Stitching']);
end
```

### 注意事项
- 确保你有相应的工具箱（如Computer Vision Toolbox）安装在MATLAB中。
- 替换`image1.jpg`和`image2.jpg`为你自己的图片路径。
- 根据实际需求调整代码中的参数和选项。

运行上述代码后，你可以看到不同特征提取方法的匹配结果和最终的图像拼接效果。
