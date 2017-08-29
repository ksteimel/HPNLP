# Tips for using Scikit-learn on very large datasets
### Author: Kenneth Steimel

## Sparsity
When constructing n-gram models, one often creates an extremely sparse feature value matrix. This is simply a result of patterns that occur throughout language: large numbers of elements occur very infrequently while a small number of elements occur with great frequency.    
One could simplify things by naively lopping off some number of features: the frequencies in a large amount of text are likely to be approximately the same as the frequencies that naturally occur in other texts so these infrequent features probably don't generalize well anyways. For example, one could only grab the top 1000 most frequent features and then use only those. However, there are more sophisticated methods that can be used like Chi-squared, PCA and SVD. Scikit-learn provides highly optimized, easily parallelizable versions of these algorithms. In addition, scikit-learn allows you to easily tune these algorithms to produce the best results in conjunction with the classifier being used. Therefore, it does make sense to keep the sparsity in there.    
However, several issues emerge as a result. 
- The large number of empty values may cause your data to consume an inordinate amount of memory.
- Many numerical libraries, like numpy, require contiguous memory. Even if you have enough memory to hold the data structure, it may not be contiguous and you may get out of memory errors.
- Filling in this array may take an incredibly long amount of time since the number of feature entries that need to be checked is just so massive.
- Common serialization methods used may fail when confronted with a large amount of data

The solution that works best for me is to use the scipy.sparse module to generate spaarse matricies. These can be utilized by scikit-learn directly. The format best for filling in values in the feature value matrix is [scipy.sparse.lil_matrix](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.lil_matrix.html#scipy.sparse.lil_matrix). However, scikit-learn works best with objects of [scipy.sparse.csr](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.csr_matrix.html#scipy.sparse.csr_matrix) format. What works best for me is to fill in values using lil_matrix and then convert to csr using .tocsr(). 


## Persistance
Ideally, you wouldn't want to recompute the features each time you trained an algorithm. This is even worse when dealing with a large amount of data. One of the most common and easiest ways to serialize the feature value matrix you've tirelessly created is by exporting the data structure you've created as a pickle. This is commonly done using the following syntax:
	import pickle
	obj = generateObject() //Pretend this function creates some complex object
	with open('myfile.pkl', 'rb') as fp
		pickle.dump(obj, fp)

See the [documentation](https://docs.python.org/3.6/library/pickle.html#pickle.HIGHEST_PROTOCOL) for more information.

However, with very large objects (more than 4 GB), problems emerge where the protocol used cannot handle objects of thsi size. If your data is sparse, you can use one of the sparse object formats mentioned above. If this is still too large, you should specify the highest protocol parameter to [pickle.load and pickle.dump](https://docs.python.org/3.6/library/pickle.html#pickle-protocols).

