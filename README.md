# Nesting_Related_Resources

Nesting Related Ressources with PHP
>
>                        --------------------------------------------------------------
 >                        |                  Nested Related Resources                |
  >                      --------------------------------------------------------------
>

                        ==============================================================
                                         # Understand Nested Resources
                        ==============================================================
                        - Resource: a set of data or functionality of a single type
                        - Subjects, pages, admins
                        - Nested resource: a resource inside another resource
                        - Dependent
                        - One-To-Many Relationships

                        i.e: | Subjects
                                  -Pages

                        - No direct access to pages
                        - Access pages through subjects
                        - Group pages by subject
                        - All links and redirects must work with new structure

                        ==============================================================
                                         # List pages by subject
                        ==============================================================
                        <?php
                        require_login();

                        // $id = isset($_GET['id']) ? $_GET['id'] : '1';
                        $id = $_GET['id'] ?? '1'; // PHP > 7.0
                        

                        $subject = find_subject_by_id($id);

                        //$page_set = find_all_pages(); // but there's a problem with this ..:-)
                        /* all pages will be listed, we just want pages to be listed if belong to the subject_id*/
                        // so we better use our function find_pages_by_subject_id()
                        $page_set = find_pages_by_subject_id($id); // second param not a must
                        ?>


                        ==============================================================
                                         # Use nested links
                        ==============================================================
                        So, the key thing that you need to remember about these nested
                        links is making sure that you can pass around the information
                        you need to maintain your nested state, so that you know what
                        the parent record is.
                            Sometimes you may be able to get that from the parent itself,
                        sometimes, you can get it from the foreign key of the child object,
                        sometimes you need to pass it in the URL so that you know it.
                        There's actually a fourth possibility as well, which is that if
                        we go to submit our form for our new page, we still have the subject ID,
                        and that's because it's in the form data. It's passed in in the form data,
                        so when we call URL four, we're not passing in the subject ID there,
                        it's actually down here in the form data, and that allows us to be able
                        to use it up here inside the form processing.

                        ==============================================================
                                         # Use nested redirects
                        ==============================================================
                        If it doesn't exist, then don't try and delete it, right? Do something
                        different instead, redirect. Redirect back to another page saying, hey sorry,
                        the page you just wanted to delete doesn't exist, right? So I can think of a
                        lot of ways that I could provide better error checking if I first go and look
                        for the page before I try and delete it. So that's why I picked this one.
                        So we'll save the result. So now we've counted for all of our links, all of
                        our forms, and all of our redirects. They all work now with our new nested structure.

                        ==============================================================
                                         # Add page count to each subject
                        ==============================================================
                        $page_set = find_pages_by_subject_id($id);
                        $page_count = mysqli_num_rows($page_set);
                        mysqli_free_result($page_set);
                          // count pages by subject_id
                          function count_pages_by_subject_id($subject_id, $options=[]) {
                            global $db;

                            $visible = $options['visible'] ?? false;

                            $sql = "SELECT COUNT(id) FROM pages "; // don't return the data to me, but count them instead
                            $sql .= "WHERE subject_id='" . db_escape($db, $subject_id) . "' ";
                            if($visible) {
                              $sql .= "AND visible = true ";
                            }
                            $sql .= "ORDER BY position ASC";
                            $result = mysqli_query($db, $sql);
                            confirm_result_set($result);
                            $row = mysqli_fetch_row($result);
                            mysqli_free_result($result);
                            $count = $row[0];
                            return $count; // we now get counted rows returned /not result anymore
                          }

                        ==============================================================
                                         # Scope page position by subject
                        ==============================================================
                        //$page_set = find_all_pages();
                        //$page_count = mysqli_num_rows($page_set);
                        //mysqli_free_result($page_set);

                        // count
                        $page_count = count_pages_by_subject_id($page['subject_id']);
                        // do not increment anymore, we're just editing
                        // count
                        $page_count = count_pages_by_subject_id($page['subject_id']) + 1 // it's a new record, so +1

                        ==============================================================
                                         # Update positions
                        ==============================================================
                        - Automatically manage position numbering
                        - Adding, moving, or deleting an item should adjust the positions
                          of other items
                        - Reduces item-by-item editing
                        - Add code to existing insert, update, and delete functions

                        - Add new item
                        - Delete item
                        - Move item later in the list
                        - Move item earlier in the list

                        # Add new item                      # Delete item
                        - Starting position 0               - Starting position: provided
                        - Ending position is provided       - Ending position: 0
                        - No existing record to consider    - Existing record to exclude from update
                        - Add 1 to all items greater than   - Subtract 1 from all items greater than
                          ending position                     starting position

                        # Move Item Later in the List
                        - Starting position is provided
                        - Ending position is provided
                        - Existing record to exclude from update
                        - Subtract 1 from all items greater than starting position and
                          less than or equal to ending position

                        # Move item earlier in the list
                        - Starting position is provided
                        - Ending position is provided
                        - Existing record to exclude from update
                        - Add 1 to all items greater than or equal to ending position
                          and less than starting position

                        function shift_subject_positions($start_pos, $end_pos, $current_id=0) {
                            if($start_pos == 0){
                                // new item, +1 to items greater than $end_pos
                            } elseif($end_pos == 0) {
                                // delete item, -1 from items greater than start $start_pos
                            } elseif($start_pos < $end_pos) {
                                // move later, -1 from items between (including $end_pos)
                            } elseif($start_pos > $send_pos) {
                                // move earlier, +1 to items between (including $end_pos)
                            }
                            // Exclude the current_id in the SQL WHERE clause
                        }

                        UPDATE subjects
                        SET position = position - 1
                        WHERE position > 2
                        AND position <= 6
                        AND id != 8;

                        - Subject positions
                        - Page positions
                        - Pages must consider the subject_id

                          // update positions
                          function shift_subject_positions($start_pos, $end_pos, $current_id=0) {
                            global $db;
                            if ($start_pos == $end_pos) { return; }

                            # QUERY_FUNCTIONS.PHP


                            $sql = "UPDATE subjects ";
                            if($start_pos == 0){
                              // new item, +1 to items greater than $end_pos
                              $sql .= "SET position = position + 1";
                              $sql .= "WHERE position >= '" . db_escape($db, $end_pos) . "' ";

                            } elseif($end_pos == 0) {
                              // delete item, -1 from items greater than start $start_pos
                              $sql .= "SET position = position - 1 ";
                              $sql .= "WHERE position > '" . db_escape($db, $start_pos) . "' ";

                            } elseif($start_pos < $end_pos) {
                              // move later, -1 from items between (including $end_pos)
                              $sql .= "SET position = position - 1 ";
                              $sql .= "WHERE position > '" . db_escape($db, $start_pos) . "' ";
                              $sql .= "AND position <= '" . db_escape($db, $end_pos);

                            } elseif($start_pos > $end_pos) {
                              // move earlier, +1 to items between (including $end_pos)
                              $sql .= "SET position = position + 1 ";
                              $sql .= "WHERE position >= '" . db_escape($db, $end_pos) . "' ";
                              $sql .= "AND position < '" . db_escape($db, $start_pos) . "' ";

                            }
                            // Exclude the current_id in the SQL WHERE clause
                            $sql .= "AND id != '" . db_escape($db, $current_id) . "' ";

                            $result = mysqli_query($db, $sql);
                            // For UPDATE statements, $result is true/false
                            if ($result) {
                              return true;
                            } else {
                              // UPDATE failed
                              echo mysqli_error($db);
                              db_disconnect($db);
                              exit();
                            }
                          }

                          # insert_subject(){...}
                              // ..update positions
                              shift_subject_positions(0, $subject['position']);

                          # update_subject(){...}
                              // update...positions
                              $old_subject = find_subject_by_id($subject['id']);
                              $old_position = $old_subject['position'];
                              shift_subject_positions($old_position, $subject['position'], $subject['id']);

                          # delete_subject(){...}
                            // use id...update position
                            $old_subject = find_subject_by_id($id);
                            $old_position = $old_subject['position'];
                            shift_subject_positions($old_position, 0, $id); // the new position is 0 because I remove it from the list

                          # Do the same for pages, but with an additional parameter
                          : subject_id - (Remember Relationship between pages & subjects tables)




>
> # Contribution
>
> - Kevin Skoglund (Instructor)
> - Stephane Sob Fouodji
> - Lynda.com
>
