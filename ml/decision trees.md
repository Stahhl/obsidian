```python
def compute_entropy(y):
    """
    Computes the entropy for 
    
    Args:
       y (ndarray): Numpy array indicating whether each example at a node is
           edible (`1`) or poisonous (`0`)
       
    Returns:
        entropy (float): Entropy at that node
        
    """
    # You need to return the following variables correctly
    entropy = 0.
    
    ### START CODE HERE ###
    if len(y) == 0:
        return entropy
    
    p1 = len(y[y == 1]) / len(y)
    
    if p1 == 0 or p1 == 1:
        return entropy
        
    entropy = -p1 * np.log2(p1) - (1- p1)*np.log2(1 - p1)  
    ### END CODE HERE ###        
    
    return entropy
```

```python
def split_dataset(X, node_indices, feature):
    """
    Splits the data at the given node into
    left and right branches
    
    Args:
        X (ndarray):             Data matrix of shape(n_samples, n_features)
        node_indices (list):     List containing the active indices. I.e, the samples being considered at this step.
        feature (int):           Index of feature to split on
    
    Returns:
        left_indices (list):     Indices with feature value == 1
        right_indices (list):    Indices with feature value == 0
    """
    
    # You need to return the following variables correctly
    left_indices = []
    right_indices = []
    
    ### START CODE HERE ###
    for i,x in enumerate(node_indices):
        if X[x][feature] == 1:
            left_indices.append(x)
        else:
            right_indices.append(x)
     
    ### END CODE HERE ###
        
    return left_indices, right_indices
```

```python
def compute_information_gain(X, y, node_indices, feature):
    
    """
    Compute the information of splitting the node on a given feature
    
    Args:
        X (ndarray):            Data matrix of shape(n_samples, n_features)
        y (array like):         list or ndarray with n_samples containing the target variable
        node_indices (ndarray): List containing the active indices. I.e, the samples being considered in this step.
   
    Returns:
        cost (float):        Cost computed
    
    """    
    # Split dataset
    left_indices, right_indices = split_dataset(X, node_indices, feature)
    
    # Some useful variables
    X_node, y_node = X[node_indices], y[node_indices]
    X_left, y_left = X[left_indices], y[left_indices]
    X_right, y_right = X[right_indices], y[right_indices]
    
    # You need to return the following variables correctly
    information_gain = 0
    
    ### START CODE HERE ###
    # Your code here to compute the entropy at the node using compute_entropy()
    node_entropy = compute_entropy(y_node)
    # Your code here to compute the entropy at the left branch
    left_entropy = compute_entropy(y_left)
    # Your code here to compute the entropy at the right branch
    right_entropy = compute_entropy(y_right)

    # Your code here to compute the proportion of examples at the left branch
    w_left = len(X_left)/len(X_node)

    # Your code here to compute the proportion of examples at the right branch
    w_right = len(X_right)/len(X_node)

    # Your code here to compute weighted entropy from the split using 
    # w_left, w_right, left_entropy and right_entropy
    weighted_entropy = w_left * left_entropy + w_right * right_entropy

    # Your code here to compute the information gain as the entropy at the node
    # minus the weighted entropy
    information_gain = node_entropy - weighted_entropy
    ### END CODE HERE ###  
    
    return information_gain
```

```python
def get_best_split(X, y, node_indices):   
    """
    Returns the optimal feature and threshold value
    to split the node data 
    
    Args:
        X (ndarray):            Data matrix of shape(n_samples, n_features)
        y (array like):         list or ndarray with n_samples containing the target variable
        node_indices (ndarray): List containing the active indices. I.e, the samples being considered in this step.

    Returns:
        best_feature (int):     The index of the best feature to split
    """    
    
    # Some useful variables
    num_features = X.shape[1]
    
    # You need to return the following variables correctly
    best_feature = -1
    
    ### START CODE HERE ###
    max_info_gain = 0
    for feature in range(num_features): 

       # Your code here to compute the information gain from splitting on this feature
       info_gain = compute_information_gain(X, y, node_indices, feature)

       # If the information gain is larger than the max seen so far
       if info_gain > max_info_gain:  
           # Your code here to set the max_info_gain and best_feature
           max_info_gain = info_gain
           best_feature = feature         
    ### END CODE HERE ##    
   
    return best_feature
```

```python
tree = []

def build_tree_recursive(X, y, node_indices, branch_name, max_depth, current_depth):
    """
    Build a tree using the recursive algorithm that split the dataset into 2 subgroups at each node.
    This function just prints the tree.
    
    Args:
        X (ndarray):            Data matrix of shape(n_samples, n_features)
        y (array like):         list or ndarray with n_samples containing the target variable
        node_indices (ndarray): List containing the active indices. I.e, the samples being considered in this step.
        branch_name (string):   Name of the branch. ['Root', 'Left', 'Right']
        max_depth (int):        Max depth of the resulting tree. 
        current_depth (int):    Current depth. Parameter used during recursive call.
   
    """ 

    # Maximum depth reached - stop splitting
    if current_depth == max_depth:
        formatting = " "*current_depth + "-"*current_depth
        print(formatting, "%s leaf node with indices" % branch_name, node_indices)
        return
   
    # Otherwise, get best split and split the data
    # Get the best feature and threshold at this node
    best_feature = get_best_split(X, y, node_indices) 
    
    formatting = "-"*current_depth
    print("%s Depth %d, %s: Split on feature: %d" % (formatting, current_depth, branch_name, best_feature))
    
    # Split the dataset at the best feature
    left_indices, right_indices = split_dataset(X, node_indices, best_feature)
    tree.append((left_indices, right_indices, best_feature))
    
    # continue splitting the left and the right child. Increment current depth
    build_tree_recursive(X, y, left_indices, "Left", max_depth, current_depth+1)
    build_tree_recursive(X, y, right_indices, "Right", max_depth, current_depth+1)
```