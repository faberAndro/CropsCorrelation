#!/usr/bin/env python
# coding: utf-8

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import ctypes
import sys
from matplotlib.colors import BoundaryNorm
from matplotlib.ticker import MaxNLocator
import pprint

ctypes.windll.shcore.SetProcessDpiAwareness(1) if 'win' in sys.platform else None   # QUESTA RIGA ASSICURA CHE L'API win RESTITUISCA LA GIUSTA DEFINIZIONE IN PIXEL
np.seterr(all='raise')


def show_2_superposed_shapes(crop_1, crop_2, color_1=None, color_2=None):

    c_1, c_2 = crops_to_indexes[crop_1], crops_to_indexes[crop_2]
    data_c_1 = data1[(data1['Item'] == crop_1) & (data1['Element'] == 'Yield')]
    data_c_2 = data1[(data1['Item'] == crop_2) & (data1['Element'] == 'Yield')]
    data_c12 = pd.merge(data_c_1, data_c_2, how='inner', on=['Year'])

    ax = plt.gca()
    plt.subplots_adjust(left=0.16)
    ax.set_ylabel('Yield [hg/ha]')
    corr = matrix_filled[c_1, c_2]
    data_c12.plot(kind='line', x='Year', y='Value_x', ax=ax, label=crop_1, c=color_1, marker='o', markersize=4)
    data_c12.plot(kind='line', x='Year', y='Value_y', ax=ax, label=crop_2, c=color_2, marker='o', markersize=4, title='correlation = %s' %round(corr, 2))
    plt.show()


def lancia_barre(m, w, labels, c='mediumaquamarine', etichette=False): # m è la matrix di dati n x n (numpy array). w è lo spessore di una barra (0..1), c: colore

    fig = plt.figure()
    ax1 = fig.add_subplot(111, projection='3d')
    # costruisce gli assi x, y e assegna il piano z=0
    n = m.shape[0]
    _x = np.arange(n)+w/2
    _y = np.arange(n)+w/2
    _xx, _yy = np.meshgrid(_x, _y)
    x_centres, y_centres = _xx.ravel(), _yy.ravel()
    bottom = np.zeros_like(x_centres)
    # assegna le etichette
    # ax1.set_xlabel('crops - [1]')
    ax1.set_xticks(_x+w/2, )
    ax1.set_xticklabels(labels, rotation=90)
    # ax1.set_ylabel('crops - [2]')
    ax1.set_yticks(_y+w/2)
    ax1.set_yticklabels(labels, rotation=90)
    ax1.set_zlim3d(0, 1)
    ax1.set_zlabel('correlation')
    ax1.set_title('crops correlation over years 1961-2018')
    # assegna lo spessore
    width = depth = w
    # assegna i valori
    z_height = np.round(m.ravel(), 2)
    # print(type(z_height))
    ax1.bar3d(x_centres, y_centres, bottom, width, depth, z_height, shade=True, color=c)
    if etichette:
        for i, z in enumerate(z_height):
            ax1.text(x_centres[i]+w/2*0.5, y_centres[i]+w/2*0.5, z+0.07, str(np.round(z, 2)), zdir=None, zorder=1)
    # print(max(z_height))
    plt.show()


def load_from_European_type_file(y):
    data = pd.read_csv(r'C:\04. IT Projects\PROJECTS\0. AI Projects\AI MISCELLANEA\Trends_Meteo_Crops\CROPS_vs_METEO_Italy\FAOSTAT_data_5-19-2020.csv', header = 0)
    data1 = data[['Element', 'Item', 'Year', 'Unit', 'Value', 'Flag Description']]  # extracts only the useful columns 
    information_required = data1['Element'] == 'Yield'
    crops = list(map(sorted(list(set(data1['Item']))).__getitem__, y))
    n_years = len(set(data1['Year']))
    n_crops = len(crops)
    return n_crops, crops, data1, information_required, n_years


def correlation_Builder():
    matrix = np.zeros((n_crops, n_crops))
    n_available_years = np.zeros((n_crops, n_crops))
    indexes_to_crops = {}
    crops_to_indexes = {}
    serie = []
    for i, crop in enumerate(crops):
        selezione_coltura = data1['Item'] == crop
        data_n = data1[information_required & selezione_coltura]
        data_n.set_index('Year', inplace=True)
        serie_n = pd.Series(data_n['Value'])
        serie.append(serie_n)
        indexes_to_crops[str(i)] = crop
        crops_to_indexes[crop] = i
    for x, s1 in enumerate(serie[:-1]):    # costruisce la matrix delle correlazioni, con le etichette: si può ridurre a metà!
        for y, s2 in enumerate(serie[x+1:]):
            # se i dati non sono correlabil il coeff. di correlazione viene lasciato a 0. Si può aggiungere una matrix che tenga conto di quali dati sono stati analizzati
            if len(s1)>lim_inf and len(s2)>lim_inf:
                # viene calcolata l'intersezione degli anni disponibili su entrambi
                ixs = s1.index.intersection(s2.index)
                s1_x = s1.loc[ixs]
                s2_x = s2.loc[ixs]
                try:
                    c_coef = s1_x.corr(s2_x)
                    matrix[x][y+x+1] = c_coef if not np.isnan(c_coef) else 0
                    n_available_years[x][y+x+1] = len(s1_x)
                except FloatingPointError:
                    if False:
                        print('warning encountered when correlating %s with %s' % (x, y))
                        print('serie:', serie )
                        print(s1, s2)
                        print('s1_x: ', s1_x)
                        print('s2_x: ', s2_x)
            else:
                matrix[x][y+x+1] = 0

    matrix_c = np.abs(matrix)
    # RIEMPE LA MATRICE TRIANGOLARE
    matrix_filled = matrix.copy()
    for i in range(len(crops)):
        matrix_filled[i][i] = 1
        for j in range(i):
            matrix_filled[i][j] = matrix_filled[j][i]
    return matrix, matrix_c, matrix_filled, n_available_years, crops_to_indexes, indexes_to_crops


if __name__=='__main__':

    # TODO: explain how to read the half meshed graph imagining the bouncing line

    # TODO: convert names to English
    # TODO: organize crops in similarity groups

    # TODO: find out why we get some 'Nan' from correlation computation. (at x=7, y=41-7)
    # TODO: check why IDE was showing issues when importing "lancia_barre" from local folder
    # TODO: fix division by zero warning
    # TODO: try simplifying using a pandas correlation matrix instead of step by step series

    lim_inf = 9   # if a time series has less than 'lim_inf' values, it is not considered.
    verbose = False
    visualization_treshold = 30
    _3_graphs = False

    i1 = input('98 types of crop are avialable. Please select a subset of them or simply type enter to capture all: ')
    crops_sub_set = list(range(20, 50))
    # crops_sub_set = list(range(15, 30))
    # crops_sub_set = [20, 21, 22, 69, 70, 71]

    n_crops, crops, data1, information_required, n_years = load_from_European_type_file(crops_sub_set)
    matrix, matrix_c, matrix_filled, n_available_years, crops_to_indexes, indexes_to_crops = correlation_Builder()
    print(indexes_to_crops)

    if _3_graphs:
        show_2_superposed_shapes('Eggplants (aubergines)', 'Hemp tow waste', 'forestgreen', 'lime')
        show_2_superposed_shapes('Maize', 'Fruit, fresh nes', 'magenta', 'darkmagenta')
        show_2_superposed_shapes('Garlic', 'Chestnut', 'dimgray', 'silver')
        exit(0)

    while True:
        # <editor-fold desc="CAPTURE USER INPUT">
        barre = pair = scatter = singola_crop = toni_a_scacchi = False
        i2 = input('What graph would you like to see (s = single, c = pair, m = mesh grid, v = variable size mesh, b = 3D bars, x = Exit) ? ')
        if i2 == 's':
            singola_crop = True
        elif i2 == 'c':
            pair = True
        elif i2 == 'm':
            toni_a_scacchi = True
        elif i2 == 'v':
            scatter = True
        elif i2 == 'b':
            barre = True
        elif i2 == 'x':
            exit(0)
        # </editor-fold>

        # <editor-fold desc="GRAFICI">
        if barre:
            lancia_barre(matrix_c, 0.5, labels=crops, etichette=True)

        if singola_crop:
            desired_crop = input('Which crop would you like to compare with all the rest? ')
            if desired_crop in crops:
                crop_index = crops_to_indexes[desired_crop]
                fig2 = plt.figure()
                sp1 = fig2.add_subplot(111)
                sp1.set_title(desired_crop + ' : correlations coeffs.')
                correlazioni = matrix_filled[crop_index, :]
                associazioni = dict(zip(crops, correlazioni))
                coppie = dict(sorted(associazioni.items(), key=lambda x: x[1]))
                c_values = np.asarray(list((coppie.values())))

                # data_color = [x/len(c_values) for x in np.arange(len(c_values))]
                c_values_mapped_to_01 = c_values/2 + 0.5 # mapping [-1, +1] to [0, 1]
                data_color = c_values_mapped_to_01
                my_cmap = plt.cm.get_cmap('PiYG')
                colors = my_cmap(data_color)
                print(data_color, colors)
                sp1.barh(list(coppie.keys()), c_values, color=colors)
                plt.show()

        if pair:
            memo = input('Would you like reviewing the crop list (y/n) ? ')
            if memo == 'y':
                print(crops)
            c1 = input('Crop 1 name? ')
            c2 = input('Crop 2 name? ')
            # extract the 2 series from the database:
            if (c1 in crops) and (c2 in crops):
                show_2_superposed_shapes(c1, c2)

        if toni_a_scacchi:

            plt.rcParams['xtick.bottom'] = plt.rcParams['xtick.labelbottom'] = False
            plt.rcParams['xtick.top'] = plt.rcParams['xtick.labeltop'] = True

            n = len(crops)
            x = y = np.arange(n+1)
            xg, yg = np.meshgrid(x, y, sparse=False, indexing='ij')
            z = matrix

            fig, (ax0) = plt.subplots(1, 1, sharey=True)
            cmap = plt.get_cmap('bwr')
            levels = MaxNLocator(nbins=10).tick_values(z.min(), z.max())
            norm = BoundaryNorm(levels, ncolors=cmap.N, clip=True)

            im = ax0.pcolormesh(xg, yg, z, cmap='PiYG')   # previous color map: RdYlGn
            fig.colorbar(im, ax=ax0)

            ax0.set_xticks(x+0.5)
            ax0.set_yticks(y+0.5)

            if len(crops_sub_set) > visualization_treshold:
                ax0.set_yticklabels(list(range(len(crops_sub_set))))
                ax0.set_xticklabels(list(range(len(crops_sub_set))), rotation=90)
            else:
                ax0.set_yticklabels(crops)
                ax0.set_xticklabels(crops, rotation=90)

            plt.show()
            plt.rcParams['xtick.bottom'] = plt.rcParams['xtick.labelbottom'] = True
            plt.rcParams['xtick.top'] = plt.rcParams['xtick.labeltop'] = False

        if scatter:

            plt.rcParams['xtick.bottom'] = plt.rcParams['xtick.labelbottom'] = False
            plt.rcParams['xtick.top'] = plt.rcParams['xtick.labeltop'] = True

            n = len(crops)
            x = y = np.arange(n)
            xg, yg = np.meshgrid(x, y, sparse=False, indexing='ij')
            z = matrix
            dimension = n_available_years*int(100/n)
            dimension = np.abs(dimension.astype(int))

            fig, (ax0) = plt.subplots(1, 1, sharey=True)
            cmap = plt.get_cmap('bwr')
            levels = MaxNLocator(nbins=10).tick_values(z.min(), z.max())
            norm = BoundaryNorm(levels, ncolors=cmap.N, clip=True)

            im = ax0.scatter(xg, yg, marker='s', s=dimension, c=z, cmap='PiYG')   # previous color map: RdYlGn
            fig.colorbar(im, ax=ax0)

            ax0.set_xticks(x)
            ax0.set_yticks(y)

            if len(crops_sub_set) > visualization_treshold:
                ax0.set_yticklabels(list(range(len(crops_sub_set))))
                ax0.set_xticklabels(list(range(len(crops_sub_set))), rotation=90)
            else:
                ax0.set_yticklabels(crops)
                ax0.set_xticklabels(crops, rotation=90)

            plt.show()
            plt.rcParams['xtick.bottom'] = plt.rcParams['xtick.labelbottom'] = True
            plt.rcParams['xtick.top'] = plt.rcParams['xtick.labeltop'] = False

        # </editor-fold>

        if verbose:
            print(len(crops))
            print(len(x))
            print(matrix.shape)
            for i, crop in enumerate(crops):
                print(i, crop)
