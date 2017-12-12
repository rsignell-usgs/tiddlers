Short on disk space? Compress those netcdf3 files using netcdf4 with deflation instead of gzipping! With ROMS and other ocean model output, my experience has been that deflation at least cuts the file size in half, and if you are willing to sacrifice a bit of precision (say you don't need currents saved to be more accurate than 0.001 m/s, for example), you can save even more. If you have large masked regions (land in ROMS, clouds in remote sensing imagery), you can save even more. We reduced some AVHRR data by a factor of 20!  

Also chunking can make user access much more performant for time series extractions.  Most data is arranged so that extraction of the complete field at a give time step is fast, but extraction of time series at a point is slow.  By rechunking you can make it much faster for time series extraction while mimimally impacting extraction of complete fields.  See Russ Rew's excellent articles on chunking: [Chunking Data: Why it Matters](https://www.unidata.ucar.edu/blogs/developer/entry/chunking_data_why_it_matters) and [Chunking Data: Choosing Shapes](https://www.unidata.ucar.edu/blogs/developer/entry/chunking_data_choosing_shapes).

Here are a three ways you can deflate and chunk your netcdf3 files into netcdf4 (or rechunk/redeflate netcdf4):

1. **[nc3tonc4](https://unidata.github.io/netcdf4-python)** (Jeff Whitaker): `conda install -c conda-forge netcdf4`

I like this tool because it gives you the ability to specify the number of significant digits, which can improve compression by quite a bit. For example, to keep 3 digits (0.001m/s) accuracy on velocity, and 4 on temperature, just do:
```
nc3tonc4 --quantize='u=3,v=3,temp=4' netcdf3.nc netcdf4.nc
```

2. **[nccopy](http://www.unidata.ucar.edu/software/netcdf/docs/netcdf/nccopy.html)** (Unidata): `conda install -c conda-forge libnetcdf`

With versions higher than netCDF-4.1.2, you can do deflation and chunking.
For example, to convert netCDF-3 data to netCDF-4
data compressed at deflation level 1 and using 10x20x30 chunks for
variables that use (time,lon,lat) dimensions:

```
nccopy -d1 -c time/10,lon/20,lat/30 netcdf3.nc netcdf4.nc
```

3. **[ncks](http://nco.sourceforge.net/)** (Charlie Zender):  `conda install -c conda-forge nco`
Let's you specify chuck sizes along any dimension:
```
ncks -4 -L 1  --cnk_dmn lat,50 --cnk_dmn lon,50 -O netcdf3.nc netcdf4.nc
```
