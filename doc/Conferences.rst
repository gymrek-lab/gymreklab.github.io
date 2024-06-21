.. _conferences:

Attending Conferences
=====================

Policies in our lab
~~~~~~~~~~~~~~~~~~~
* The lab can fund one major conference per year per student
* Before submitting any abstract or paper please be sure to first get Melissa's approval and that of any co-authors on the work
* If you are able to obtain funding help (e.g. travel awards available through GEPA or elsewhere) you may also attend an additional meeting. In many cases existing travel awards will not cover everything and I might be able to cover the difference, but you should check with Melissa first

    Finally, regarding finding funding for meetings: there are many options available, but you might need to do some digging to find them. GEPA in UCSD has a lot of information about many conferences available for students. In many cases, financial assistance is prioritized for students who show an emphasis on leadership and/or how the research gives back to the community. Beyond GEPA, some conferences will have specific travel awards available. Some departments (e.g. CSE) may also sometimes offer travel awards. We can work on compiling a list of these as a group.

* Conferences that are local and therefore do not require travel/lodging costs generally do not count toward the above, but again you should check with Melissa first before registering
* Generally, to attend a conference you must submit an abstract and plan to present your work. This requirement is waived if it is your first year in the lab, in which case you likely would not have had enough time to have something to present yet

Some conferences we attend
~~~~~~~~~~~~~~~~~~~~~~~~~~
* `American Society of Human Genetics (ASHG) <https://www.ashg.org/meetings>`_: this is the biggest human genetics meeting of the year. This conference is very broad and is relevant to almost all of our work. This meeting is huge (thousands of participants) which is good and bad. It is a big honor and great opportunity for exposure to give a talk at ASHG. Many people announce their latest and greatest new work and technologies here. On the other hand, because ASHG is so huge it can be hard to meet new people and can be a bit overwhelming.
* `Biology of Genomes <https://meetings.cshl.edu/meetings.aspx?meet=GENOME>`_: This is a much smaller meeting held at Cold Spring Harbor in Long Island NY. It is not all genetics and includes some broader genomics applications. My opinion is that everyone should get to a Cold Spring Harbor meeting at least once as a trainee. The setting is much more intimate than ASHG, where everyone is stuck on the CSHL campus (which is quite pretty) for several days and there are many more opportunities to meet new people.
* `RECOMB <https://recomb.org>`_: RECOMB is much more methods and computational biology focused. You might remember when RECOMB was held at San Diego a couple years ago. It is a strong community of people who really care about methods details and I always end up learning something new and coming away with new methods ideas at this meeting. It typically alternates between the US and non-US locations every other year.

Other options specific to our field are
* `AGBT <https://www.agbt.org/events/general-meeting>`_ which has more of a technology flavor
* `Genome Informatics <https://meetings.cshl.edu/meetings.aspx?meet=info>`_ which is more methods focused but maybe a bit less so than RECOMB
* `Gordon Research Conferences (GRC) <https://www.grc.org>`_ which are held on very specific topics and typically occur in ski-resort type settings closer to the CSHL feel
* The `computational genomics summer institute (CGSI) <http://computationalgenomics.bioinformatics.ucla.edu>`_ at UCLA

There are also meetings that are more general (not just bioinformatics) but you should also consider attending:
* `SACNAS: Society for the Advancement of Chicanos/Hispanics and Native Americans in Science <https://projects.iq.harvard.edu/sacnasharvard/what-sacnas>`_. All are welcome, but with emphasis on Hispanic / Latinx students
* `ABRCMS: Annual Biomedical Research Conference for Minoritized Scientists <https://www.abrcms.org>`_. All Welcome, emphasis on African American students

Travel and reimbursement guidelines
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
To get reimbursed for conference-related expenses, please refer to these guidelines from Dorit:

https://docs.google.com/document/d/1v414-VEuYpOuHidq2AlqxrvjSTfj16QdpaXrVsSOi0U

Before your upcoming trip or conference, please take a moment to review the UCSD policies regarding `Travel Highlights <https://blink.ucsd.edu/travel/_files/TravelPolicyHighlights.pdf>`_.

If you wish to submit independently for reimbursement use SAP Concur and follow `this link <https://support.ucsd.edu/services?id=kb_article_view&sysparm_article=KB0032014>`_ for more info on how to do it.

If you prefer for Dorit to submit your reimbursement report, please follow these steps:

**Before the business trip:**

1. `Add Dorit as your delegate <https://support.ucsd.edu/finance?id=kb_article_view&sys_kb_id=287c8670dba5d8104cd8f06e0f9619d1>`_ on your SAP Concur account. 

2. Email Dorit the trip/conference details as soon you know them for submission: `Travel Request <https://support.ucsd.edu/finance?id=kb_article_view&sysparm_article=KB0032013&sys_kb_id=0edbfb231b2c711048e9cae5604bcb98&table=kb_knowledge>`_

    Remember, the request must be pre-authorized by our Financial Unit *before* the trip.

**Important Information for your trip:**

1. *Meals & Tips*: While traveling for business within the continental US for 1-29 days, you can be reimbursed for meal and tip expenses up to a daily maximum of $79.

*Lodging*: Hotel stays can be reimbursed up to $275 per night, excluding taxes and mandatory hotel fees. If you choose to share a room with another lab member, only the lab member whose name appears on the receipt may submit the receipt for reimbursement. More information about meals and lodging expenses can be found `here <https://blink.ucsd.edu/travel/travel-policy/meals-lodging/index.html>`_.

2. *Receipts*: Keep all receipts for meals, lodging, and other travel expenses.

3. *Bookings*: Ensure your hotel and flight reservations are booked in your name.

**After your trip:**

1. Please organize your receipts in the `reimbursement report <https://docs.google.com/spreadsheets/d/1gJxdq_XuJDynoe1ogz0oXi4LKm_Wp4tgGrSjdEPevM0>`_

2. Scan your receipts separately, save them as PDF/JPG, and send them to Dorit.

As always, let Dorit know if you have any questions.


Creating a poster
~~~~~~~~~~~~~~~~~
Here are some helpful tips!

1. Always check the size requirements before starting! It's generally a good idea to specify these dimensions as the canvas size in your graphics software
2. Leave at least 0.5 inches of padding on all sides of your poster just in case the printer requires it
3. When placing figures onto your poster, try to use vector image formats (.svg, .pdf, etc) rather than raster formats (.png, .jpg, etc).

    .. figure:: https://github.com/gymrek-lab/gymreklab.github.io/assets/23412689/4f1a241a-f47f-4702-8719-76026161f31c
        :alt: Raster vs vector images
        :align: center
        :width: 400px

    a. Raster is the traditional format that you're probably familiar with. It stores colors for each pixel in your image. By contrast, vector formats store each component of your figure as an object. For example, a line in your image will be defined in vector format by a start position, end position, and color -- rather than a series of black pixels.
    b. If you place a raster image on your poster, there's a good chance it will appear blurry when printed. The advantage of vector formats is that they can be rescaled to any arbitrary size and will never appear blurry!
    c. If you use matplotlib or pandas to create your figures, you can easily just change the desired output filename from ".png" to ".pdf" to create a vector version of the figure.

    .. warning::
        Some images (like Manhattan plots) will have so many objects in them that Adobe Illustrator will freeze and crash when you try to load them. For situations like these, it's best to import them as PNG. To minimize blurrines, you can try to recreate the figure with a high DPI (or PPI) and then resize it down within Illustrator.

4. When creating your poster, try to use software that will allow you to work with vector (as opposed to raster) images. So don't use google slides/drawings! Adobe illustrator is probably the best option. You can ask Dorit to get you a license. After you're done, export your poster as a PDF rather than a PNG.
5. You can find some old lab posters in `the lab's Google Drive <https://drive.google.com/drive/folders/1ora8McmJShuJeiwb1hCSrsKWEiMoAxCs>`_.

    .. note::
        Please consider uploading your poster here after you're done with it so that future years can look back on it and glean wisdom! Also, never think that your poster isn't good enough to be shared here! The best way to communicate an idea will always depend on its content, after all. You never know who might be inspired by the design of your poster one day.

6. Our logos can be found in `the lab Google Drive <https://drive.google.com/drive/folders/1-egL2EVfTh7wH4wmfFcruGtJMplnPVQQ>`_. For UCSD's, you can refer to `this Jacobs School of Engineering webpage <https://jacobsschool.ucsd.edu/logos>`_. Also, consider displaying your email and a QR code link to your GitHub repo or documentation.
7. The cheapest place to print posters is probably on campus at the print shop at `UCSD Campus Curbside Pickup <https://maps.app.goo.gl/FseyUa62wk3Qztu5A>`_. You can request reimbursement as part of your conference expenses afterwards.
    a. Go to `their online portal <https://ucsdimprints.myprintdesk.net/DSF/SmartStore.aspx?6xni2of2cF2gL05u6lNHBp6AwVlPfgDQIgaPc5Cokq4RKYVvn2cx3C2V0adSszgU#!/CategoryHome/9>`_ to create an order and submit a PDF of your poster. (Use `this link <https://blink.ucsd.edu/facilities/tritonprint/index.html>`__ to navigate to the portal if the former doesn't work.)
    b. After logging in, click on "Signs and Banners" and then "BUY NOW" under the category: "POSTERS, CHARTS, AND DISPLAYS".

        .. figure:: https://github.com/gymrek-lab/gymreklab.github.io/assets/23412689/efd10f1d-c2d6-42ab-a97f-57eb1a8d79af
            :alt: Navigating the online print shop portal
            :align: center
            :width: 400px

    c. Make sure to specify the right number of pages, the size, and the media (recommended: 36 Lb Heavyweight Coated Bond):

        .. figure:: https://github.com/gymrek-lab/gymreklab.github.io/assets/23412689/3f794299-7690-4f1a-b9f0-4e2c9dc067e1
            :alt: Poster print settings 1
            :align: center
            :width: 400px

        .. figure:: https://github.com/gymrek-lab/gymreklab.github.io/assets/23412689/08a5faad-43ed-4a27-ac76-629821288bb4
            :alt: Poster print settings 2
            :align: center
            :width: 400px

    d. After submitting the order, call them to ask when to pick it up.
