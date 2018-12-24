# HoloViews tips

This is my ongoing compilation of HoloViews snippets to accomplish particular tasks. One day this will actually be decently organized.

## Legends

`my_dataset.to(hv.DesiredElement).overlay('a_kdim')` will automatically generate a legend

In the case that you are concatenating two plots through the `*` operator, for example, you can add legends like:

```python
(
    hv.Element(data, kdims=[blah], vdims=[blah], label='the first legend entry')
    * hv.Element(data, kdims=[blah], vdims=[blah], label='the second legend entry')
)
```

## Operating in a declarative, hv.Dataset-based regime

Using the `.to()` declarative syntax: plotting multiple Gaussians of different sigma with `pd.DataFrame`s and `hv.Dataset`s. This is a demonstration of how to visualize `DataFrame`-based data in HoloViews

Procedure:
* Calculate the pdfs
* Put into a `DataFrame`
* Make the `DataFrame` tidy by melting
* Create an `hv.Dataset` where the independent vars are the x coordinate and the sigma parameter and the dependent value is the pdf at an x
* Plot an arbitrary HoloViews plot using `dataset.to(hv.ElementType, kdims=['independent_var(s)'], vdims=['dependent_var(s)']`
* If you don't specify all the kdims in the dataset as arguments to the `.to()` method, they will be used to generate an `hv.HoloMap`. 
* If you want to instead overlay curves with different values of an independent var (e.g., the sigma parameter), use the `.overlay('your_overlay_kdim')

```python
# #####################
# data generation

def calc_normal_pdf(x: np.ndarray, mu=0, sigma=1):
    return 1 / np.sqrt(2 * np.pi * sigma ** 2) * np.exp(- 0.5 * (x - mu) ** 2 / sigma ** 2)

x = np.linspace(-5, 5, 41)

sigmas = np.logspace(0, 2, 10)
normal_pdfs = [calc_normal_pdf(x, sigma=sigma) for sigma in sigmas]

# #####################
# visualization

pdf_df = pd.DataFrame(
    {
        'x': x,
        **{
            f'Normal, σ={sigma:.2f}': pdf 
            for sigma, pdf in zip(sigmas, normal_pdfs)
        }
    }
).melt(id_vars='x', var_name='distribution', value_name='pdf')

pdf_hvds = hv.Dataset(pdf_df, kdims=['x', 'distribution'], vdims='pdf')

(
    pdf_hvds.to(hv.Curve, kdims='x').overlay('distribution', sort=False)
    *
    pdf_hvds.to(hv.Scatter, kdims='x').overlay('distribution', sort=False)
)
```

## Organizing plot keys

Disable string sorting

`hv_dataset.to(hv.Thing).overlay('my_kdim', sort=False)`
