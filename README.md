
This notebook aims to show how easly we could convert a raster file into a polygon, extracting or reclassifying the raster to certain values before the conversion

##### Import module


```python
import gdal
```

##### Hint: To open a raster with gdal- just use that


```python
gdal_data = gdal.Open(r"PAN_alt.tif")
gdal_band = gdal_data.GetRasterBand(1)

```

##### Importing path


```python
path = r"PAN_alt.tif"
out_file = path.replace(".tif", "_recl.tif")
```

##### Cheking the polygonize options


```python
!gdal_polygonize.py
```

    
    gdal_polygonize [-8] [-nomask] [-mask filename] raster_file [-b band|mask]
                    [-q] [-f ogr_format] out_file [layer] [fieldname]
    


##### Checking the calculator utility


```python
!gdal_calc.py
```

    Usage: gdal_calc.py --calc=expression --outfile=out_filename [-A filename]
                        [--A_band=n] [-B...-Z filename] [other_options]
    
    Options:
      -h, --help            show this help message and exit
      --calc=expression     calculation in gdalnumeric syntax using +-/* or any
                            numpy array functions (i.e. log10())
      -A filename           input gdal raster file, you can use any letter (A-Z)
      --A_band=n            number of raster band for file A (default 1)
      --outfile=filename    output file to generate or fill
      --NoDataValue=value   output nodata value (default datatype specific value)
      --type=datatype       output datatype, must be one of ['Byte', 'UInt16',
                            'Int16', 'UInt32', 'Int32', 'Float32', 'Float64']
      --format=gdal_format  GDAL format for output file
      --creation-option=option, --co=option
                            Passes a creation option to the output format driver.
                            Multiple options may be listed. See format specific
                            documentation for legal creation options for each
                            format.
      --allBands=[A-Z]      process all bands of given raster (A-Z)
      --overwrite           overwrite output file if it already exists
      --debug               print debugging information
      --quiet               suppress progress messages


#### Our try will be to use a basic raster calculation routine to calculate a new raster where the following values will replace some ranges in the digital elevation model

1 = 0 - 1000

2 = 1001 - 2000

3 = 2001 - 3000

4 = 3001 - 4000


```python
!gdal_calc.py -A "PAN_alt.tif" --outfile "PAN_alt_recl.tif" --calc="(logical_and(0<A,A<=1000)*1) + (logical_and(1001<A, A<=2000)*2) + (logical_and(2001<A, A<=4000)*4)" --NoDataValue=255
```

    0.. 1.. 3.. 4.. 6.. 7.. 9.. 11.. 12.. 14.. 15.. 17.. 19.. 20.. 22.. 23.. 25.. 26.. 28.. 30.. 31.. 33.. 34.. 36.. 38.. 39.. 41.. 42.. 44.. 46.. 47.. 49.. 50.. 52.. 53.. 55.. 57.. 58.. 60.. 61.. 63.. 65.. 66.. 68.. 69.. 71.. 73.. 74.. 76.. 77.. 79.. 80.. 82.. 84.. 85.. 87.. 88.. 90.. 92.. 93.. 95.. 96.. 98.. 100 - Done


##### Visualizing the results


```python
!gdal_polygonize.py "PAN_alt_recl.tif" -f "ESRI Shapefile" PAN_elev_range.shp 
```

    Creating output PAN_elev_range.shp of format ESRI Shapefile.
    0...10...20...30...40...50...60...70...80...90...100 - done.



```python
import geopandas as gpd
import matplotlib.pyplot as plt
```


```python
fig, ax = plt.subplots(1, figsize = (20, 15))
ax.set_aspect('equal')
shp = gpd.GeoDataFrame.from_file(r"PAN_elev_range.shp")
shp.plot(ax=ax, color='white', edgecolor='black', cmap = "jet")
plt.legend(shp["DN"], ["a", "b", "c"])
```

    /home/roger/anaconda3/envs/GIS/lib/python3.6/site-packages/matplotlib/legend.py:798: UserWarning: Legend does not support 2 instances.
    A proxy artist may be used instead.
    See: http://matplotlib.org/users/legend_guide.html#creating-artists-specifically-for-adding-to-the-legend-aka-proxy-artists
      "aka-proxy-artists".format(orig_handle)
    /home/roger/anaconda3/envs/GIS/lib/python3.6/site-packages/matplotlib/legend.py:798: UserWarning: Legend does not support 1 instances.
    A proxy artist may be used instead.
    See: http://matplotlib.org/users/legend_guide.html#creating-artists-specifically-for-adding-to-the-legend-aka-proxy-artists
      "aka-proxy-artists".format(orig_handle)
    /home/roger/anaconda3/envs/GIS/lib/python3.6/site-packages/matplotlib/legend.py:798: UserWarning: Legend does not support 0 instances.
    A proxy artist may be used instead.
    See: http://matplotlib.org/users/legend_guide.html#creating-artists-specifically-for-adding-to-the-legend-aka-proxy-artists
      "aka-proxy-artists".format(orig_handle)





    <matplotlib.legend.Legend at 0x7f45a5bd0588>




![png](05_Converting_Contour_Curves_to_Polygons_files/05_Converting_Contour_Curves_to_Polygons_17_2.png)



```python
shp
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>DN</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>POLYGON ((-83.2 9.666666668, -83.1916666670000...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>POLYGON ((-79.56666681199999 9.641666668999999...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>POLYGON ((-82.55000002600001 9.566666671999998...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>POLYGON ((-82.51666669400001 9.533333339999999...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2</td>
      <td>POLYGON ((-83.19166666700001 9.575000005, -83....</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0</td>
      <td>POLYGON ((-82.49166669500001 9.516666674, -82....</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0</td>
      <td>POLYGON ((-82.525000027 9.491666674999999, -82...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0</td>
      <td>POLYGON ((-82.500000028 9.491666674999999, -82...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0</td>
      <td>POLYGON ((-82.425000031 9.458333343, -82.41666...</td>
    </tr>
    <tr>
      <th>9</th>
      <td>2</td>
      <td>POLYGON ((-82.941666677 9.441666676999999, -82...</td>
    </tr>
    <tr>
      <th>10</th>
      <td>0</td>
      <td>POLYGON ((-82.266666704 9.441666676999999, -82...</td>
    </tr>
    <tr>
      <th>11</th>
      <td>0</td>
      <td>POLYGON ((-82.941666677 9.433333343999999, -82...</td>
    </tr>
    <tr>
      <th>12</th>
      <td>2</td>
      <td>POLYGON ((-82.791666683 9.416666677999999, -82...</td>
    </tr>
    <tr>
      <th>13</th>
      <td>0</td>
      <td>POLYGON ((-82.908333345 9.400000012, -82.90000...</td>
    </tr>
    <tr>
      <th>14</th>
      <td>0</td>
      <td>POLYGON ((-79.825000135 9.400000012, -79.81666...</td>
    </tr>
    <tr>
      <th>15</th>
      <td>1</td>
      <td>POLYGON ((-82.291666703 9.441666676999999, -82...</td>
    </tr>
    <tr>
      <th>16</th>
      <td>2</td>
      <td>POLYGON ((-83.2 9.400000012, -83.1916666670000...</td>
    </tr>
    <tr>
      <th>17</th>
      <td>0</td>
      <td>POLYGON ((-83.19166666700001 9.383333345999999...</td>
    </tr>
    <tr>
      <th>18</th>
      <td>0</td>
      <td>POLYGON ((-83.000000008 9.375000012999999, -82...</td>
    </tr>
    <tr>
      <th>19</th>
      <td>0</td>
      <td>POLYGON ((-82.95833334300001 9.358333346999999...</td>
    </tr>
    <tr>
      <th>20</th>
      <td>1</td>
      <td>POLYGON ((-82.233333372 9.358333346999999, -82...</td>
    </tr>
    <tr>
      <th>21</th>
      <td>1</td>
      <td>POLYGON ((-82.95833334300001 9.350000013999999...</td>
    </tr>
    <tr>
      <th>22</th>
      <td>0</td>
      <td>POLYGON ((-82.250000038 9.350000013999999, -82...</td>
    </tr>
    <tr>
      <th>23</th>
      <td>0</td>
      <td>POLYGON ((-82.125000043 9.325000014999999, -82...</td>
    </tr>
    <tr>
      <th>24</th>
      <td>0</td>
      <td>POLYGON ((-82.208333373 9.316666681999999, -82...</td>
    </tr>
    <tr>
      <th>25</th>
      <td>0</td>
      <td>POLYGON ((-82.16666670800001 9.316666681999999...</td>
    </tr>
    <tr>
      <th>26</th>
      <td>1</td>
      <td>POLYGON ((-82.891666679 9.308333349, -82.88333...</td>
    </tr>
    <tr>
      <th>27</th>
      <td>0</td>
      <td>POLYGON ((-82.20000004000001 9.300000015999998...</td>
    </tr>
    <tr>
      <th>28</th>
      <td>1</td>
      <td>POLYGON ((-82.175000041 9.291666682999999, -82...</td>
    </tr>
    <tr>
      <th>29</th>
      <td>0</td>
      <td>POLYGON ((-82.183333374 9.266666683999999, -82...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>216</th>
      <td>2</td>
      <td>POLYGON ((-78.216666866 7.708333412999999, -78...</td>
    </tr>
    <tr>
      <th>217</th>
      <td>1</td>
      <td>POLYGON ((-81.633333396 7.700000079999999, -81...</td>
    </tr>
    <tr>
      <th>218</th>
      <td>2</td>
      <td>POLYGON ((-78.158333535 7.750000077999999, -78...</td>
    </tr>
    <tr>
      <th>219</th>
      <td>2</td>
      <td>POLYGON ((-78.041666873 7.675000080999999, -78...</td>
    </tr>
    <tr>
      <th>220</th>
      <td>1</td>
      <td>POLYGON ((-81.35000007400001 7.658333415, -81....</td>
    </tr>
    <tr>
      <th>221</th>
      <td>1</td>
      <td>POLYGON ((-81.708333393 7.650000081999999, -81...</td>
    </tr>
    <tr>
      <th>222</th>
      <td>1</td>
      <td>POLYGON ((-80.000000128 7.633333415999999, -79...</td>
    </tr>
    <tr>
      <th>223</th>
      <td>2</td>
      <td>POLYGON ((-77.708333553 7.975000068999999, -77...</td>
    </tr>
    <tr>
      <th>224</th>
      <td>2</td>
      <td>POLYGON ((-78.01666687399999 7.583333417999999...</td>
    </tr>
    <tr>
      <th>225</th>
      <td>1</td>
      <td>POLYGON ((-81.216666746 7.575000084999999, -81...</td>
    </tr>
    <tr>
      <th>226</th>
      <td>2</td>
      <td>POLYGON ((-78.000000208 7.550000085999999, -77...</td>
    </tr>
    <tr>
      <th>227</th>
      <td>2</td>
      <td>POLYGON ((-77.52500022700001 7.600000083999999...</td>
    </tr>
    <tr>
      <th>228</th>
      <td>2</td>
      <td>POLYGON ((-77.983333542 7.633333415999999, -77...</td>
    </tr>
    <tr>
      <th>229</th>
      <td>1</td>
      <td>POLYGON ((-81.108333417 7.583333417999999, -81...</td>
    </tr>
    <tr>
      <th>230</th>
      <td>2</td>
      <td>POLYGON ((-80.716666766 7.500000087999999, -80...</td>
    </tr>
    <tr>
      <th>231</th>
      <td>2</td>
      <td>POLYGON ((-77.916666878 7.500000087999999, -77...</td>
    </tr>
    <tr>
      <th>232</th>
      <td>1</td>
      <td>POLYGON ((-82.241666705 7.483333421999999, -82...</td>
    </tr>
    <tr>
      <th>233</th>
      <td>2</td>
      <td>POLYGON ((-77.925000211 7.475000089, -77.91666...</td>
    </tr>
    <tr>
      <th>234</th>
      <td>2</td>
      <td>POLYGON ((-80.633333436 7.358333427, -80.62500...</td>
    </tr>
    <tr>
      <th>235</th>
      <td>2</td>
      <td>POLYGON ((-80.658333435 7.333333428, -80.65000...</td>
    </tr>
    <tr>
      <th>236</th>
      <td>1</td>
      <td>POLYGON ((-81.750000058 7.650000081999999, -81...</td>
    </tr>
    <tr>
      <th>237</th>
      <td>2</td>
      <td>POLYGON ((-80.758333431 7.341666760999999, -80...</td>
    </tr>
    <tr>
      <th>238</th>
      <td>2</td>
      <td>POLYGON ((-80.691666767 7.366666759999999, -80...</td>
    </tr>
    <tr>
      <th>239</th>
      <td>2</td>
      <td>POLYGON ((-80.758333431 7.308333428999999, -80...</td>
    </tr>
    <tr>
      <th>240</th>
      <td>2</td>
      <td>POLYGON ((-80.78333343 7.291666762999999, -80....</td>
    </tr>
    <tr>
      <th>241</th>
      <td>2</td>
      <td>POLYGON ((-80.775000097 7.283333429999999, -80...</td>
    </tr>
    <tr>
      <th>242</th>
      <td>1</td>
      <td>POLYGON ((-81.825000055 7.291666762999999, -81...</td>
    </tr>
    <tr>
      <th>243</th>
      <td>1</td>
      <td>POLYGON ((-81.808333389 7.225000098999999, -81...</td>
    </tr>
    <tr>
      <th>244</th>
      <td>1</td>
      <td>POLYGON ((-81.800000056 7.216666765999999, -81...</td>
    </tr>
    <tr>
      <th>245</th>
      <td>1</td>
      <td>POLYGON ((-83.2 9.699999999999999, -82.8000000...</td>
    </tr>
  </tbody>
</table>
<p>246 rows × 2 columns</p>
</div>


