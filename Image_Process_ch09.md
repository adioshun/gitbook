## Chapter 9. Geometric Camera Models

### 9.1 2D Projective Transformation
찍혀진 방향, 거리에 따라 기울어진, 찌그러진 모양을 변환 하기 

![](https://qph.ec.quoracdn.net/main-qimg-7f88e97748213f7a3fb8959afda744ac?convert_to_webp=true)


* Hierarchy of Transformations
  * Projective Transformations
  * Affine Transformations : A subgroup of P L(3) consisting of matrices having last row (0, 0, 1) is called an affine group.
  * Similarity Transformations : Similarity transformations are a subset of affine transformations.
  * Euclidean Transformations : Euclidean transformations are a subset of similarity transformations.
  * Isometry Transformation : Orientation preserving mappling

![](http://homepages.inf.ed.ac.uk/rbf/HIPR2/figs/affhei.gif)

### 9.2 Conversion of Transformations

#### 2D Projective Distortion (시점 변경?)
![](http://cfile24.uf.tistory.com/image/2604EB41577D2C44289430)
좌에서 찍은 사진을 우에서 찍은 형태로 변경 
#### 3D Projective Distortion

#### Radial Distortion (휘지 않은것으로?)
볼록렌즈로 사진 찍은것 같은것을 평면 모양으로 변경 
![](http://hugin.sourceforge.net/docs/manual/Barrel-distortion.jpg)


