# passnetwork-repo
A program that can help analyse passing links and networks among players, also looking at their average locations and formation structures of teams.

# Import Libraries

    import re
    import json
    import pandas as pd
    import numpy as np
    from mplsoccer import VerticalPitch, FontManager
    import matplotlib.pyplot as plt
    import matplotlib.patheffects as path_effects
    from matplotlib.colors import to_rgba
    from highlight_text import ax_text

# Setting Up Data

    teamidhome = int(input('Enter Team ID'))   #Look for TeamID from WhoScored.com HTML Tags
    df_home = df_1[df_1['teamId']==teamidhome]
    df_home['passer'] = df_home['playerId']
    df_home['reciever'] = df_home['playerId'].shift(-1)
    passes_home = df_home[df_home['eventType']== 'Pass']
    successful_home = df_home[df_home['outcomeType']=='Successful']#successful passes DF
    
    teamidaway = int(input('Enter Team ID other team'))    #Look for TeamID from       WhoScored.com HTML Tags
    df_away = df_2[df_2['teamId']==teamidaway]
    df_away['passer'] = df_away['playerId']
    df_away['reciever'] = df_away['playerId'].shift(-1)
    passes_away = df_away[df_away['eventType']== 'Pass']
    successful_away = df_away[df_away['outcomeType']=='Successful'] #succesful passes DF

     passes_home = passes_home.merge(players_df[["playerId", "name"]], on='playerId', how='left')
    passes_away = passes_away.merge(players_df[["playerId", "name"]], on='playerId', how='left')
    successful_home = successful_home.merge(players_df[["playerId", "isFirstEleven"]], on='playerId', how='left')
    successful_home = successful_home[successful_home['isFirstEleven'] == True]
    successful_away = successful_away.merge(players_df[["playerId", "isFirstEleven"]], on='playerId', how='left')
    successful_away = successful_away[successful_away['isFirstEleven'] == True]
    
    avg_loc_home = successful_home.groupby('playerId').agg({'x':['mean'],'y':['mean','count']})
    avg_loc_home.columns = ['x','y','count']
    avg_loc_home = avg_loc_home.merge(players_df[['playerId', 'name', 'shirtNo', 'position']],on='playerId', how='left')
    
    avg_loc_away = successful_away.groupby('playerId').agg({'x':['mean'],'y':['mean','count']})
    avg_loc_away.columns = ['x','y','count']
    avg_loc_away = avg_loc_away.merge(players_df[['playerId', 'name', 'shirtNo', 'position']],on='playerId', how='left')

    pass_between_home = successful_home.groupby(['passer', 'reciever']).id.count().reset_index()
    pass_between_home.rename({'id': 'pass_count'}, axis='columns', inplace=True)
    pass_between_home = pass_between_home.merge(avg_loc_home, left_on='passer', right_on='playerId')
    
    pass_between_home = pass_between_home.merge(avg_loc_home, left_on='reciever', right_on='playerId',
                                                    suffixes=['', '_end'])
    pass_between_away = successful_away.groupby(['passer', 'reciever']).id.count().reset_index()
    pass_between_away.rename({'id': 'pass_count'}, axis='columns', inplace=True)
    pass_between_away = pass_between_away.merge(avg_loc_away, left_on='passer', right_on='playerId')
    
    pass_between_away = pass_between_away.merge(avg_loc_away, left_on='reciever', right_on='playerId',
                                                    suffixes=['', '_end'])

    #filtering for min4 passes in between players
    pass_between_home = pass_between_home[pass_between_home['pass_count'] >4]
    pass_between_away = pass_between_away[pass_between_away['pass_count'] >4]


# Plotting the Visual

    #Specify the URL or local path to the Oswald font file
    oswald_font_url = "https://raw.githubusercontent.com/google/fonts/main/ofl/oswald/Oswald%5Bwght%5D.ttf"

    #Create the FontManager instance
    oswald_regular = FontManager(oswald_font_url)

    TEAM1 = input("ENTER TEAM1 NAME")
    TEAM2 = input("ENTER TEAM1 NAME")


    #Define your parameters
    MAX_LINE_WIDTH = 500
    MAX_MARKER_SIZE = 1500
    MIN_TRANSPARENCY = 0.0

    #Calculate line width and marker size based on your data
    pass_between_home['width'] = (pass_between_home.pass_count / pass_between_home.pass_count.max() * MAX_LINE_WIDTH)
    avg_loc_home['marker_size'] = (avg_loc_home['count'] / avg_loc_home['count'].max() * MAX_MARKER_SIZE)

    #Calculate color and transparency
    color = np.array(to_rgba('purple'))
    color = np.tile(color, (len(pass_between_home), 1))
    c_transparency = pass_between_home.pass_count / pass_between_home.pass_count.max()
    c_transparency = (c_transparency * (1 - MIN_TRANSPARENCY)) + MIN_TRANSPARENCY
    color[:, 3] = c_transparency



    #Create a VerticalPitch object
    pitch = VerticalPitch(
        pitch_type="opta",
        pitch_color="white",
        line_color="black",
        linewidth=1,
    )

    fig, axs = pitch.grid(ncols=2,title_height=0.08, endnote_space=0,
                          # Turn off the endnote/title axis. I usually do this after
                          # I am happy with the chart layout and text placement
                          axis=False,
                      t    itle_space=0, grid_height=0.82, endnote_height=0.05)

    #Plot the pass network
    arrows = pitch.arrows(
        pass_between_home.x,
        pass_between_home.y,
        pass_between_home.x_end,
        pass_between_home.y_end,
        lw=c_transparency,
        color=color,
        zorder=2,
        ax=axs['pitch'][0],
    )
    pass_nodes = pitch.scatter(
        avg_loc_home.x,
        avg_loc_home.y,
        color="red",
        edgecolors="black",
        s=avg_loc_home.marker_size,
        linewidth=0.5,
        alpha=1,
        ax=axs['pitch'][0],
        )

    for index, row in avg_loc_home.iterrows():
        text = pitch.annotate(
            row.shirtNo,
            xy=(row.x, row.y),
            c="white",
            va="center",
            ha="center",
            size=12,
            weight="bold",
            ax=axs['pitch'][0],
            fontproperties=oswald_regular.prop,
        )
        text.set_path_effects([path_effects.withStroke(linewidth=1, foreground="yellow")])

    #2nd Team Pass Network Plots Start

    # Define your parameters
    MAX_LINE_WIDTH = 500
    MAX_MARKER_SIZE = 1500
    MIN_TRANSPARENCY = 0.0

    # Calculate line width and marker size based on your data
    pass_between_away['width'] = (pass_between_away.pass_count / pass_between_away.pass_count.max() * MAX_LINE_WIDTH)
    avg_loc_away['marker_size'] = (avg_loc_away['count'] / avg_loc_away['count'].max() * MAX_MARKER_SIZE)

    # Calculate color and transparency
    color1 = np.array(to_rgba('purple'))
    color1 = np.tile(color1, (len(pass_between_away), 1))
    c_transparency1 = pass_between_away.pass_count / pass_between_away.pass_count.max()
    c_transparency1 = (c_transparency1 * (1 - MIN_TRANSPARENCY)) + MIN_TRANSPARENCY
    color1[:, 3] = c_transparency1


    # Plot the pass network
        arrows = pitch.arrows(
            pass_between_away.x,
            pass_between_away.y,
            pass_between_away.x_end,
            pass_between_away.y_end,
            lw=c_transparency1,
            color=color1,
            zorder=2,
            ax=axs['pitch'][1],
            )
        pass_nodes = pitch.scatter(
                avg_loc_away.x,
                avg_loc_away.y,
                color="skyblue",
                edgecolors="black",
                s=avg_loc_away.marker_size,
                linewidth=0.5,
                alpha=1,
                ax=axs['pitch'][1],
            )    
    
        for index, row in avg_loc_away.iterrows():
            text = pitch.annotate(
                row.shirtNo,
                xy=(row.x, row.y),
                c="white",
                va="center",
                ha="center",
                size=12,
                weight="bold",
                ax=axs['pitch'][1],
                fontproperties=oswald_regular.prop,
            )
            text.set_path_effects([path_effects.withStroke(linewidth=1, foreground="black")])
        
        # Add labels to the pass networks
        highlight_text = [{'color': 'red', 'fontproperties': oswald_regular.prop},
                          {'color': 'skyblue', 'fontproperties': oswald_regular.prop}]
        ax_text(0.5, 0.7, f"<{TEAM1}> & <{TEAM2}> Pass Networks", fontsize=28, color='#000009',
                                        fontproperties=oswald_regular.prop,highlight_textprops=highlight_text,
                                        ha='center', va='center',ax=axs['title'])
        
        axs["endnote"].text(
            1,
            1,
            "@athalakbar13 n/
            data via Opta",
            color="black",
            va="center",
            ha="right",
            fontsize=12,
            fontproperties=oswald_regular.prop,
        )
        plt.show()
