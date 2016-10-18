<!-- $theme: default -->

# OpenFOAMで二次元円柱周りの流れを見る
## Kobe univ. B4 藤原 巧
### 参考: http://opencae.gifu-nct.ac.jp/pukiwiki/index.php?plugin=attach&refer=%C2%E8%A3%B2%A3%B4%B2%F3%CA%D9%B6%AF%B2%F1%A1%A7H250810&openfile=2Dflow_around_cylinder_with_pisoFoam_rev.pdf

- - -

# 今日の目的: 円柱の$C_D$値を求める

- - -

# 本日の流れ

1. 円柱.stlファイルの作成
2. メッシュ作成(blockMesh)
3. メッシュ作成(snappyHexMesh)
4. 境界条件, 計算条件
5. 計算実行(pisoFoam)
6. 計算結果の表示(paraFoam)

- - -

# 1. 円柱(.stl)ファイルの作成

やりました ファイルあげます

- - -

#  2. メッシュ作成(blockMesh)

- - -

# baseとなるtutorialのコピー

```bash
$ cp -r ~/OpenFOAM/OpenFOAM-2.4.0/tutorials \
/incompressible/pisoFoam/ras/cavity .

$ mv cavity [dirname] # 好きな名前で e.g.) cylinder
```

- - -
`constant/polyMesh/blockMeshDict` 内 17行目以降

```
convertToMeters 0.01;

vertices
(
    (-2 -1.5 0)
    ( 4 -1.5 0)
    ( 4  1.5 0)
    (-2  1.5 0)
    (-2 -1.5 0.1)
    ( 4 -1.5 0.1)
    ( 4  1.5 0.1)
    (-2  1.5 0.1)
);

blocks
(
    hex (0 1 2 3 4 5 6 7) (60 30 1) simpleGrading (1 1 1)
);
```

- - -
# blockMeshDict設定

```
boundary
(
    upstream 
    {
        type patch;
        faces
        (
            (0 4 7 3)
        );
    }
    downstream 
    {
        type patch;
        faces
        (
		  	   (1 2 6 5)
        );
    } 
```
まだ続く

- - -
# blockMeshDict設定
```
    upANDdown 
    {
        type patch; 
        faces
        (
            (0 1 5 4)
            (3 7 6 2)
        );
    }
    frontANDback
    {
        type empty;
        faces
        (
            (0 3 2 1)
            (4 5 6 7)
        );
    }
);
```

- - -
# blockMeshの実行

0/ だとか constant/ だとか system/ だとかがあるディレクトリに`cd`

```
$ blockMesh #meshの作成
```
meshの確認をします

```
$ mv ./0 ./0.org # これをしないと表示のときにエラー
$ paraFoam #結果の確認
```
薄めの直方体が出来るはず
- - -
# 3. メッシュ作成(snappyHexMesh)
1から作るのはめんどくさいので`tutorials`から持ってくる
```bash
$ cp ~/OpenFOAM/OpenFAOM-2.4.0/tutorials/incompressible/ \
pisoFoam/les/motorBike/motorBike/system/snappyhexMeshDict .
```
```
    29	geometry
    30	{
    31		cylinder-s.stl
    32		{
    33			type triSurfaceMesh;
    34			name cylinder;
    35		}
    36	};
```
```
   103	    refinementSurfaces
   104	    {
   105			 cylinder
   106			 {
   107				 level (0 0);
   108			 }
   109	    }
```

- - -

```
 146	    locationInMesh (-0.01 0 0.0005);
```
snappyHexMeshの実行
```
$ snappyHexMesh # この時 0/が存在してはいけない
```
確認
```
$ paraFoam
```

- - -
# snappyHexMeshのmesh情報をコピー

```bash
$ cp -r 0.01/polyMesh/* constant/polyMesh/
```

```bash
$ rm -r 0.005/ 0.01/ # こいつらはもう不要
```
- - -
# 物性値と解析モデル

`constant/RASProperties` ファイル
```b
RASModel 	laminar;
turbulence	off;
```

- - -

# controlDict

`system/controlDict` の設定
```
application     pisoFoam;
startFrom       startTime;
startTime       0;
stopAt          endTime;
endTime         15;
deltaT          0.001;
writeControl    timeStep;
writeInterval   1000;
purgeWrite      0;
writeFormat     ascii;
writePrecision  6;
writeCompression off;
timeFormat      general;
timePrecision   6;
runTimeModifiable true;
```

- - -

# 0/の設定

p, U以外は不要なので削除

```bash
$ rm epsilon k nut nuTilda 
```

- - -
# 0.org/U

```
internalField   uniform (2.0E-2 0 0);

boundaryField
{
	upstream
	{
		type 		fixedValue;
		value 	uniform (2.0E-2 0 0);
	}
	downstream
	{
		type 			inletOutlet;
		inletValue 	uniform (2.0E-2 0 0);
		value 		$internalField;
	}
	upANDdown
	{
		type 			inletOutlet;
		inletValue 	uniform (2.0E-2 0 0);
		value 		$internalField;
	}

```
- - -
```
	frontANDback
	{
		type 		empty;
	}
	cylinder
	{
		type 		fixedValue;
		value 	uniform (0 0 0);
	}
}
```
- - -
# 0.org/p
```
internalField   uniform 0;

boundaryField
{
	upstream
	{
		type 		fixedValue;
		value 	0;
	}
	downstream
	{
		type 		fixedValue;
		value 	0;
	}
	upANDdown
	{
		type 		fixedValue;
		value 	0;
	}

```

- - -

```
	frontANDback
	{
		type 		empty;
	}
	cylinder
	{
		type 		zeroGradient;
	}
}
```

- - -
# 実行

```bash
$ pisoFoam > piso.log
```

```bash
$ paraFoam
```