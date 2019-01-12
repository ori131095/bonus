# bonus
def type_determination (data_list):
    first_row = data_list[0].lower().strip().split(' ')
        # determinate if file in rows or columns.
    if first_row[1][0] == 'x' or first_row[1][0] == 'y' or first_row[1][0] == 'd':
       return analyze_cols(data_list)
    else:
        return analyze_rows(data_list)

def analyze_cols(data1):
    # creates dictionary for all the data and variables of the axis
    dic = {}
    # add the b parameters to the dictionary and remove it from the data
    b_parameters = data1[-1]
    b_parameters_list = b_parameters.lower().strip().split(' ')
    dic[b_parameters_list[0]] = b_parameters_list[1], b_parameters_list[2], b_parameters_list[3]
    data1.remove(b_parameters)
    # add the a parameters to the dictionary and remove it from the data
    a_parameters = data1[-1]
    a_parameters_list = a_parameters.lower().strip().split(' ')
    dic[a_parameters_list[0]] = a_parameters_list[1], a_parameters_list[2], a_parameters_list[3]
    data1.remove(a_parameters)
    # remove the spare line before the axes
    data1 = data1[:-1]
    # add the x axis to the dictionary and remove it from the data
    x_axis = data1[-2]
    # add the x axis to the dictionary and remove it from the data
    x_axis_list = x_axis.strip().split(': ')
    dic[x_axis_list[0]] = x_axis_list[1]
    data1.remove(x_axis)
    y_axis = data1[-1]
    # add the y axis to the dictionary and remove it from the data
    y_axis_list = y_axis.strip().split(': ')
    dic[y_axis_list[0]] = y_axis_list[1]
    data1.remove(y_axis)
    # remove the spare line before the axes
    spare_line = data1[-1]
    data1.remove(spare_line)
    # creates list of x,y,dx,dy
    first_line_list = data1[0].lower().strip().split(' ')
    data1.remove(data1[0])
    # creates empty list for each column
    first_list = []
    second_list = []
    third_list = []
    forth_list = []
    #add columns to lists and check if there is length error
    try:
        for line in data1:
            line_list = line.strip().split(' ')
            first_list.append(line_list[0])
            second_list.append(line_list[1])
            third_list.append(line_list[2])
            forth_list.append(line_list[3])
    except:
        return "Input file error: Data lists are not the same length."
    # add the columns to the corresponding dictionary key
    dic[first_line_list[0]] = first_list
    dic[first_line_list[1]] = second_list
    dic[first_line_list[2]] = third_list
    dic[first_line_list[3]] = forth_list
    # check if the uncertainties values dx and dy are positive
    dx_values = dic['dx']
    dy_values = dic['dy']
    for indx in range(len(first_list)):
        if float(dx_values[indx]) < 0 or float(dy_values[indx]) < 0:
            return 'Input file error: Not all uncertainties are positive.'
    return dic

def analyze_rows(data1):
    # creates dictionary for all the data and variables of the axis
    dic = {}
    # add the b parameters to the dictionary and remove it from the data
    b_parameters = data1[-1]
    b_parameters_list = b_parameters.lower().strip().split(' ')
    dic[b_parameters_list[0]] = b_parameters_list[1], b_parameters_list[2], b_parameters_list[3]
    data1.remove(b_parameters)
    # add the a parameters to the dictionary and remove it from the data
    a_parameters = data1[-1]
    a_parameters_list = a_parameters.lower().strip().split(' ')
    dic[a_parameters_list[0]] = a_parameters_list[1], a_parameters_list[2], a_parameters_list[3]
    data1.remove(a_parameters)
    # remove the spare line before the axes
    data1=data1[:-1]
    # add the x axis to the dictionary and remove it from the data
    x_axis = data1[-2]
    x_axis_list = x_axis.strip().split(': ')
    dic[x_axis_list[0]] = x_axis_list[1]
    data1.remove(x_axis)
    y_axis = data1[-1]
    # add the y axis to the dictionary and remove it from the data
    y_axis_list = y_axis.strip().split(': ')
    dic[y_axis_list[0]] = y_axis_list[1]
    data1.remove(y_axis)
    # remove the spare line before the axes
    data1 = data1[:-1]
    # set the length of the first line as base to check for errors
    first_line_list = data1[0].strip().split(' ')
    line_length = len(first_line_list)
    # for each line determinate for what variable it belongs, and add to dictionary
    for line in data1:
        fixed_line = line.lower().strip().split(' ')
        # check if the length of the row is equal to the the base line
        if len(fixed_line) != line_length:
            return "Input file error: Data lists are not the same length."
        if fixed_line[0] == 'x':
            dic['x'] = fixed_line[1:]
        if fixed_line[0] == 'dx':
            # check if all values are positive, and if not, add them to the dictionary
            for indx in range(1, len(fixed_line)):
                if float(fixed_line[indx]) < 0:
                    return'Input file error: Not all uncertainties are positive.'
            dic['dx'] = fixed_line[1:]
        if fixed_line[0] == 'y':
            dic['y'] = fixed_line[1:]
        if fixed_line[0] == 'dy':
            # check if all values are positive, and if not, add them to the
            for indx in range(1, len(fixed_line)):
                if float(fixed_line[indx]) < 0:
                    return 'Input file error: Not all uncertainties are positive.'
            dic['dy'] = fixed_line[1:]
    return dic

def calculate_temp_chi(a, b, input_dict):
    sum_chi_sq = 0
    n = len(input_dict['x'])
    for index in range(n):
        yi = float(input_dict['y'][index])
        xi = float(input_dict['x'][index])
        dyi = float(input_dict['dy'][index])
        temp_value = ((yi - (a * xi + b)) / dyi) ** 2
        sum_chi_sq += temp_value
    return sum_chi_sq

# this function calculate the value of chi^2.
def calculate_chi_sq(input_dict):
    min_a = min(float(input_dict['a'][0]),float(input_dict['a'][1]))
    max_a = max(float(input_dict['a'][0]),float(input_dict['a'][1]))
    step_a = abs(float(input_dict['a'][2]))
    min_b = min(float(input_dict['b'][0]),float(input_dict['b'][1]))
    max_b = max(float(input_dict['b'][0]),float(input_dict['b'][1]))
    step_b = abs(float(input_dict['b'][2]))
    min_chi = calculate_temp_chi(min_a, min_b, input_dict)
    chi_min_a = min_a
    chi_min_b = min_b
    b = min_b
    while b <= max_b:
        a = min_a
        while a <= max_a:
            temp_chi = calculate_temp_chi(a, b, input_dict)
            if temp_chi < min_chi:
                min_chi = temp_chi
                chi_min_a = a
                chi_min_b = b
            a += step_a
        b += step_b
    return [chi_min_a, step_a, chi_min_b, step_b, min_chi]

def search_best_parameter (filename):
    file2 = open(filename, 'r')
    data = file2.readlines()
    input_dic = type_determination(data)
    eval_parameters = calculate_chi_sq(input_dic)
    print(input_dic)
    return eval_parameters

print (search_best_parameter('inputbonus.txt'))
